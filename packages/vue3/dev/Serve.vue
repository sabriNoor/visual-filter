<script>
export default {
  name: "Serve",
  data() {
    return {
      currentFilter: null,
      resetFilterTrigger: 0,
      isInitialLoad: true,
      undoStack: [],
      redoStack: [],
      filteringOptions: {
        data: [
          {
            name: "First Name",
            type: "nominal",
            values: ["Obada", "Ahmad", "Omar"],
          },
          {
            name: "Last Name",
            type: "nominal",
            values: ["Khalili", "Drhili", "Hala hili"],
          },
          {
            name: "Grade",
            type: "numeric",
            values: [3.72, 3.52, 3.4],
          },
        ],
        methods: {
          numeric: {
            "="(cellValue, argument) {
              return cellValue == argument
            },
            ">"(cellValue, argument) {
              return cellValue > argument
            },
            "<"(cellValue, argument) {
              return cellValue < argument
            },
          },
          nominal: {
            contains(cellValue, argument) {
              return cellValue.includes(argument)
            },
            startsWith(cellValue, argument) {
              return cellValue.startsWith(argument)
            },
            endsWith(cellValue, argument) {
              return cellValue.endsWith(argument)
            },
          },
        },
      },
    }
  },
  watch:{
    currentFilter:{
      deep:true,
      handler(newFilter){
        localStorage.setItem("lastFilter", JSON.stringify(newFilter))
      }
    }
  },
  methods: {
    captureFilterUpdate(ctx) {
      const newFilterStr = JSON.stringify(ctx.filter)
      const currentFilterStr = JSON.stringify(this.currentFilter)

      if (newFilterStr !== currentFilterStr) {
        this.undoStack.push(currentFilterStr)
        if (this.undoStack.length > 20) this.undoStack.shift()
        this.redoStack = []
        this.currentFilter = ctx.filter
      }
    },
    saveFilter() {
      if (!this.currentFilter) {
        alert("No filter to save!")
        return
      }
      const name = prompt("Enter a name for this filter:")
      if (!name) return
      localStorage.setItem(`savedFilter-${name}`, JSON.stringify(this.currentFilter))
      const url = `${window.location.origin}${window.location.pathname}?filterName=${encodeURIComponent(name)}`
      alert(`Filter saved! Share this URL: ${url}`)
    },
    loadSavedFilterFromUrl(){
      const params = new URLSearchParams(window.location.search)
      const filterName = params.get("filterName")
      if (filterName) {
        const savedFilter = localStorage.getItem(`savedFilter-${filterName}`)
        if (savedFilter) {
          try {
            const parsedFilter = JSON.parse(savedFilter)
            this.currentFilter = parsedFilter
          } catch (e) {
            console.warn("Failed to parse saved filter from URL:", e)
          }
        }
      }
    },
    redoFilter(){
      this.undoStack.push(this.currentFilter ? JSON.stringify(this.currentFilter) : null)
      if (this.undoStack.length > 20) {
        this.undoStack.shift()
      }
      const nextFilter = this.redoStack.pop()
      this.currentFilter = nextFilter ? JSON.parse(nextFilter) : null
    },
    undoFilter(){
      this.redoStack.push(this.currentFilter ? JSON.stringify(this.currentFilter) : null)
      if (this.redoStack.length > 20) {
        this.redoStack.shift()
      }
      const previousFilter = this.undoStack.pop()
      this.currentFilter = previousFilter ? JSON.parse(previousFilter) : null
    },
    loadLastFilter(){
      const savedFilter = localStorage.getItem("lastFilter")
      if (savedFilter) {
        try {
          const parsedFilter = JSON.parse(savedFilter)
          this.currentFilter = parsedFilter
        } catch (e) {
          console.warn("Failed to parse saved filter:", e)
        }
      }
    },
    clearFilter(){
      this.currentFilter = null
      this.resetFilterTrigger += 1
      localStorage.removeItem("lastFilter")
    },
  },

  mounted(){
    this.loadLastFilter()

    this.loadSavedFilterFromUrl()

  }
}
</script>

<template>
  <VueVisualFilter
    :filtering-options="filteringOptions"
    :filter-value="currentFilter"
    :reset-filter-trigger="resetFilterTrigger"
    @filter-update="captureFilterUpdate"
  />
  <button @click="saveFilter">Save Filter</button>
  <button @click="clearFilter"> Clear Filter </button>
  <button @click="undoFilter" :disabled="undoStack.length === 0"> Undo </button>
  <button @click="redoFilter" :disabled="redoStack.length === 0"> Redo </button>

</template>
