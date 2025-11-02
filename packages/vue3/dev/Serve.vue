<script>
export default {
  name: "Serve",
  data() {
    return {
      currentFilter: null,
      canRedo: false,
      canUndo: false,
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
      this.canRedo = ctx.canRedo
      this.canUndo = ctx.canUndo
      if(ctx.action !== 'set'){
        this.currentFilter = ctx.filter
      }
      console.log(`filter updated via ${ctx.action}:`, ctx.filter)
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
            this.$refs.filter.setFilter(parsedFilter)
          } catch (e) {
            console.warn("Failed to parse saved filter from URL:", e)
          }
        }
      }
    },
    redoFilter(){
      this.$refs.filter.redo()
    },
    undoFilter(){
      this.$refs.filter.undo()
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
      this.$refs.filter.setFilter(this.currentFilter)
    },
    clearFilter(){
      this.$refs.filter.clearFilter()
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
    ref="filter"
    :filtering-options="filteringOptions"
    @filter-update="captureFilterUpdate"
  />
  <button @click="saveFilter">Save Filter</button>
  <button @click="clearFilter"> Clear Filter </button>
  <button @click="undoFilter" :disabled="!canUndo"> Undo </button>
  <button @click="redoFilter" :disabled="!canRedo"> Redo </button>

</template>
