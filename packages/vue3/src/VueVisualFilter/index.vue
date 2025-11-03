<script>
import { h } from "vue"
import {
  FilterType,
  GroupType,
  DataType,
  deepCopy,
} from "@visual-filter/common"
import applyFilter from "@visual-filter/applyer"

import FilterGroup from "./FilterGroup.vue"
import FilterCondition from "./FilterCondition.vue"

export default {
  name: "VueVisualFilter",
  emits: ["filterUpdate"],
  props: {
    filteringOptions: {
      type: Object,
      required: true,
      validator(value) {
        try {
          return (
            value.data.length &&
            value.data.every(
              (field, index, fields) =>
                typeof field.name === "string" &&
                typeof field.type === "string" &&
                field.values.constructor === Array &&
                (index > 0
                  ? field.values.length === fields[index - 1].values.length
                  : true),
            ) &&
            Object.values(value.methods.numeric).every(
              (method) => typeof method === "function",
            ) &&
            Object.values(value.methods.nominal).every(
              (method) => typeof method === "function",
            )
          )
        } catch {
          return false
        }
      },
    },
  },

  data() {
    return {
      filter: {
        type: FilterType.GROUP,
        groupType: GroupType.AND,
        filters: [],
      },
      undoStack: [],
      redoStack: [],
      maxStackSize: 20,
      action: null,
      prevFilterSnapshot: null,
    }
  },

  computed: {
    fieldNames() {
      return this.filteringOptions.data.map((field) => field.name)
    },
    numericMethodNames() {
      return Object.keys(this.filteringOptions.methods.numeric)
    },
    nominalMethodNames() {
      return Object.keys(this.filteringOptions.methods.nominal)
    },
    canRedo() {
      return this.redoStack.length > 0
    },
    canUndo() {
      return this.undoStack.length > 0
    },
  },

  mounted() {
    this.prevFilterSnapshot = deepCopy(this.filter)
  },

  watch: {
    filter: {
      deep: true,
      handler(newFilter) {
        const newSnapshot = JSON.stringify(newFilter)
        const oldSnapshot = JSON.stringify(this.prevFilterSnapshot)

        if (newSnapshot !== oldSnapshot) {
          this.pushToHistory(newFilter)
          this.emitUpdate()
        }
      },
    },
  },

  methods: {
    pushToStack(stack, item) {
      stack.push(item)
      if (stack.length > this.maxStackSize) {
        stack.shift()
      }
    },

    pushToHistory(newFilter) {
      this.pushToStack(this.undoStack, deepCopy(this.prevFilterSnapshot))
      this.redoStack = []
      this.prevFilterSnapshot = deepCopy(newFilter)
    },

    setFilter(newFilter) {
      this.action = "set"
      this.pushToHistory(newFilter)
      this.filter = deepCopy(newFilter)
      this.emitUpdate()
    },

    undo() {
      if (!this.canUndo) return
      this.action = "undo"
      const prev = this.undoStack.pop()
      this.pushToStack(this.redoStack, deepCopy(this.filter))
      this.prevFilterSnapshot = deepCopy(prev)
      this.filter = deepCopy(prev)
      this.emitUpdate()
    },

    redo() {
      if (!this.canRedo) return
      this.action = "redo"
      const next = this.redoStack.pop()
      this.pushToStack(this.undoStack, deepCopy(this.filter))
      this.prevFilterSnapshot = deepCopy(next)
      this.filter = deepCopy(next)
      this.emitUpdate()
    },

    clearFilter() {
      this.action = "clear"
      const newFilter = {
        type: FilterType.GROUP,
        groupType: GroupType.AND,
        filters: [],
      }
      this.pushToHistory(newFilter)
      this.filter = newFilter
      this.emitUpdate()
    },

    emitUpdate() {
      this.$emit("filterUpdate", {
        filter: deepCopy(this.filter),
        data: applyFilter(
          this.filter,
          this.filteringOptions.methods,
          deepCopy(this.filteringOptions.data),
        ),
        action: this.action || "update",
        canUndo: this.canUndo,
        canRedo: this.canRedo,
      })
      this.action = null
    },

    updateConditionField(condition, newFieldName) {
      const {
        type: newType,
        values: [newSampleValue = ""],
      } = this.filteringOptions.data.find(
        (field) => field.name === newFieldName,
      )
      if (condition.dataType !== newType) {
        condition.method =
          (newType === DataType.NUMERIC
            ? this.numericMethodNames[0]
            : this.nominalMethodNames[0]) || ""
        condition.argument = newSampleValue
        condition.dataType = newType
      }
    },

    addFilter(filters, newFilterType) {
      this.action = "add"
      if (newFilterType === FilterType.GROUP) {
        filters.push({
          type: FilterType.GROUP,
          groupType: GroupType.AND,
          filters: [],
        })
      } else {
        const {
          name,
          type,
          values: [sampleValue = ""],
        } = this.filteringOptions.data[0]

        filters.push({
          type: FilterType.CONDITION,
          fieldName: name,
          dataType: type,
          method:
            (type === DataType.NUMERIC
              ? this.numericMethodNames[0]
              : this.nominalMethodNames[0]) || "",
          argument: sampleValue,
        })
      }
    },

    deleteFilter(filterToDelete) {
      this.action = "delete"
      const recursiveDeletion = (filter, index, filters) => {
        if (filter === filterToDelete) {
          filters.splice(index, 1)
        } else if (filter.type === FilterType.GROUP) {
          filter.filters.forEach(recursiveDeletion)
        }
      }

      if (filterToDelete !== this.filter) {
        recursiveDeletion(this.filter)
      }
    },
  },

  render() {
    const createVisualizer = (filter) => {
      if (filter.type === FilterType.GROUP) {
        return h(
          FilterGroup,
          {
            group: filter,
            filterTypes: Object.values(FilterType),
            groupTypes: Object.values(GroupType),
            removable: filter !== this.filter,
            onAddFilter: this.addFilter,
            onDeleteGroup: this.deleteFilter,
          },
          {
            groupTypes: this.$slots.groupTypes,
            filterAddition: this.$slots.filterAddition,
            groupDeletion: this.$slots.groupDeletion,
            groupChildren: () => filter.filters.map(createVisualizer),
          },
        )
      } else {
        return h(
          FilterCondition,
          {
            condition: filter,
            fieldNames: this.fieldNames,
            numericMethodNames: this.numericMethodNames,
            nominalMethodNames: this.nominalMethodNames,
            onUpdateField: this.updateConditionField,
            onDeleteCondition: this.deleteFilter,
          },
          {
            fieldUpdation: this.$slots.fieldUpdation,
            methodUpdation: this.$slots.methodUpdation,
            argumentUpdation: this.$slots.argumentUpdation,
            conditionDeletion: this.$slots.conditionDeletion,
          },
        )
      }
    }

    return createVisualizer(this.filter)
  },
}
</script>
