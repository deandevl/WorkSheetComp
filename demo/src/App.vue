<template>
  <div class="demo_container">
    <work-sheet-comp
        :sheet_rows="sheet_rows"
        :headings="headings"
        :column_widths="column_widths"
        :table_width="table_width"
        :rows_per_view="rows_per_view"
        :css_variables="css_variables"
        v-on:work_sheet_comp_cell="show_cell">
    </work-sheet-comp>
  </div>
</template>

<script>
  import WorkSheetComp from 'worksheetcomp';

  export default {
    name: 'App',
    data: function() {
      return {
        sheet_rows: null,
        headings: [
          'Index',
          'Parameter_1','Big_Heading_Parameter_2','Parameter_3','Parameter_4','Parameter_5',
          'Parameter_6','Parameter_7','Parameter_8','Parameter_9','Parameter_10',
          'Parameter_11','Parameter_12','Parameter_13','Parameter_14','Parameter_15'
        ],

        column_widths: [
          60,
          100,120,80,140,120,
          120,140,100,60,120,
          130,120,100,80,110
        ],

        table_width: 800,
        rows_per_view: 25,

        css_variables: {
          worksheet_comp_border_color: 'white',
          worksheet_comp_table_color: 'white',
          worksheet_comp_background_color: 'black',
          worksheet_comp_row_hover_border: 'solid 1px white'
        }
      };
    },
    components: {
      WorkSheetComp
    },
    methods: {
      show_cell: function(obj){
        console.log(`row_idx: ${obj.row} col_idx: ${obj.col}`);
      }
    },
    mounted: function() {
      this.sheet_rows = [];
      for(let i = 0; i < 100; i++) {
        const row = [];
        row.push(i+1);
        for(let ii = 0; ii < 15; ii++){
          const val = (Math.random() * 100.0) + 1.0;
          row.push(+val.toFixed(2));
        }
        this.sheet_rows.push(row);
      }
      //assign strings to rows
      this.sheet_rows[0][4] = new Date('5/24/1949').toDateString();
      this.sheet_rows[1][4] = 'A String';
    }
  }
</script>

<style>
  .demo_container {
    height: 40rem;
    width: 60rem;
    background-color: #000000;
    padding: 2rem;
  }
</style>