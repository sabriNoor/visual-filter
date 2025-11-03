<script>
export default {
  name: "Serve",
  data() {
    return {
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
          {
            name: "Enrollment Date",
            type: "date",
            values: ["2020-09-01", "2019-09-01", "2021-09-01"],
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
          date:{
            before(cellValue, argument) {
              return new Date(cellValue) < new Date(argument)
            },
            after(cellValue, argument) {
              return new Date(cellValue) > new Date(argument)
            },
            on(cellValue, argument) {
              return new Date(cellValue).toDateString() === new Date(argument).toDateString()
            },
            between(cellValue, argument1, argument2) {
              const date = new Date(cellValue)
              return date >= new Date(argument1) && date <= new Date(argument2)
            }
          },
        },
      }
    }
  },
  methods: {
    captureFilterUpdate(ctx) {
      console.log(ctx)
    },
  },
}
</script>

<template>
  <VueVisualFilter
    :filtering-options="filteringOptions"
    @filter-update="captureFilterUpdate"
  />
</template>
