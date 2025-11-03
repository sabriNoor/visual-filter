<script>
import { DataType } from "@visual-filter/common"

export default {
  name: "FilterCondition",
  emits: ["updateField", "deleteCondition"],
  props: {
    condition: {
      type: Object,
      required: true,
      validator(value) {
        return value.constructor === Object
      },
    },
    fieldNames: {
      type: Array,
      required: true,
    },
    numericMethodNames: {
      type: Array,
      required: true,
    },
    nominalMethodNames: {
      type: Array,
      required: true,
    },
    dateMethodNames: {
      type: Array,
      required: true,
    },
  },
  computed: {
    isNumeric() {
      return this.condition.dataType === DataType.NUMERIC
    },
    methodNames() {
      if (this.condition.dataType === DataType.NUMERIC) {
        return this.numericMethodNames
      } else if (this.condition.dataType === DataType.NOMINAL) {
        return this.nominalMethodNames
      } else if (this.condition.dataType === DataType.DATE) {
        return this.dateMethodNames
      } else {
        console.log("Unknown data type:", this.condition.dataType)
        console.log("Available data types:", DataType)
        return []
      }
    },
  },
  methods: {
    updateField(newFieldName) {
      if (this.fieldNames.includes(newFieldName)) {
        this.$emit("updateField", this.condition, newFieldName)
      }
    },
  },
}
</script>

<template>
  <div class="space-x-2">
    <slot name="fieldUpdation" v-bind="{ fieldNames, condition, updateField }">
      <select
        v-model="condition.fieldName"
        @change="updateField($event.target.value)"
        data-testId="field-name-select"
      >
        <option v-for="field in fieldNames" :key="field" :value="field">
          {{ field }}
        </option>
      </select>
    </slot>
    <slot
      name="methodUpdation"
      v-bind="{
        methodNames,
        condition,
      }"
    >
      <select v-model="condition.method" data-testId="method-select">
        <option
          v-for="method in methodNames"
          :key="method"
          :value="method"
        >
          {{ method }}
        </option>
      </select>
    </slot>
    <slot name="argumentUpdation" :condition="condition">
      <input
        type="text"
        v-model="condition.argument"
        data-testId="argument-input"
      />
      
    </slot>
    <slot name="argumentExtra" :condition="condition">
      <input
        v-if="condition.dataType === 'date' && condition.method === 'between'"
        type="text"
        v-model="condition.argument2"
        placeholder="and"
        data-testId="argument2-input"
      />
    </slot>
    <slot
      name="conditionDeletion"
      :deleteCondition="() => $emit('deleteCondition', condition)"
    >
      <button @click="$emit('deleteCondition', condition)" data-testId="remove-condition-button">x</button>
    </slot>
  </div>
</template>
