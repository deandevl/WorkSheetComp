<template>
  <div class="worksheetComp">
    <div class="worksheetComp_idxBox" v-if="sheet_rows !== null && headings !== null" :style="column_styles[0]">
      <table class="worksheetComp_table">
        <tbody>
        <tr>
          <th class="worksheetComp_th" v-on:click="sort_column(0)">{{headings[0]}}</th>
        </tr>
        </tbody>
      </table>
    </div>

    <div id="hor" class="worksheetComp_headingsBox" v-if="sheet_rows !== null && headings !== null" :style="hor_box_style">
      <table class="worksheetComp_table">
        <tbody>
        <tr>
          <th
              class="worksheetComp_th"
              v-for="cell_idx in headings.length-1"
              :key="cell_idx"
              :style="column_styles[cell_idx]"
              v-on:click="sort_column(cell_idx)">
            {{headings[cell_idx]}}
          </th>
        </tr>
        </tbody>
      </table>
    </div>

    <div id="vert" class="worksheetComp_vertBox" v-if="sheet_rows !== null" :style="vert_box_style">
      <table class="worksheetComp_table" :style="vert_table_style">
        <tbody>
        <tr v-for="(row,row_idx) in sheet_rows" :key="row_idx">
          <td class="worksheetComp_td">
            {{row[0]}}
          </td>
        </tr>
        </tbody>
      </table>
    </div>

    <div class="worksheetComp_dataBox" v-if="sheet_rows !== null && headings !== null" :style="data_box_style"  v-on:scroll="scroll_table">
      <table class="worksheetComp_table">
        <tbody>
        <tr class="worksheetComp_trData" v-for="(row,row_idx) in sheet_rows" :key="row_idx">
          <td
              class="worksheetComp_td"
              v-for="cell_idx in headings.length-1"
              :key="cell_idx"
              :style="column_styles[cell_idx]"
              v-on:click="cell_click($event,row_idx,cell_idx)">
            {{row[cell_idx]}}
          </td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'WorkSheetComp',
    data(){
      return {
        computed_idx_box_width: 0,
        computed_table_width: 0,
        computed_table_height: 0,
        selected_td: null,
        sort_col_idx: null,
        sort_descend: null
      }
    },
    props: {
      sheet_rows: {
        type: Array,
        default: null
      },
      headings: {
        type: Array,
        default: null
      },
      column_widths: {
        type: Array,
        default: null
      },
      table_width: {
        type: Number,
        default: 600
      },
      rows_per_view: {
        type: Number,
        default: 10
      },
      sort_descending: {
        type: Boolean,
        default: false
      },
      sort_column_idx: {
        type: Number,
        default: 0
      },
      css_variables: {
        type: Object,
        default: null
      }
    },
    computed: {
      column_styles: function(){
        let total_width = 0;
        const n_cells = this.sheet_rows[0].length;
        let column_styles = new Array(n_cells);
        for(let i=0; i<n_cells; i++){
          if(this.column_widths === null || this.column_widths[i] === null){
            column_styles[i] = `min-width:${120}px;` +
              `max-width:${120}px;` +
              'overflow: hidden;cursor: pointer;';
            total_width += 120;
          }else{
            column_styles[i] = `min-width:${this.column_widths[i]}px;` +
              `max-width:${this.column_widths[i]}px;` +
              'overflow: hidden;cursor: pointer;';
            total_width += this.column_widths[i];
          }
        }

        this.computed_idx_box_width = 0;
        if(this.column_widths === null){
          this.computed_idx_box_width = 120;
        }else if(this.column_widths[0] === null){
          this.computed_idx_box_width = 120;
        }else{
          this.computed_idx_box_width = this.column_widths[0];
        }

        //each cell has column_width[i] plus 2 (1px padding left and 1px padding right)
        const table_width = total_width + (this.headings.length * 2);
        if(this.table_width > table_width){
          this.computed_table_width = table_width + 2;
        }else {
          this.computed_table_width = this.table_width;
        }
        return column_styles;
      },
      hor_box_style: function(){
        return `left:${this.computed_idx_box_width+1}px; width:${this.computed_table_width}px;`;
      },
      vert_box_style: function(){
        const n_rows = this.sheet_rows.length;
        let vert_box_height = 0;
        if(n_rows < this.rows_per_view){
          //each row is 16 pixels in height + 2 top/bottom padding
          vert_box_height = (18 * n_rows);
        }else{
          //each row is 16 pixels in height + 2 top/bottom padding
          vert_box_height = (18 * this.rows_per_view);
        }
        return `width:${this.computed_idx_box_width}px; height:${vert_box_height}px;`;
      },
      vert_table_style: function(){
        return `width:${this.computed_idx_box_width}px;`;
      },
      data_box_style: function(){
        let box_style = null;
        if(this.sheet_rows !== null){
          const n_rows = this.sheet_rows.length;
          let data_box_height = 0;
          if(n_rows < this.rows_per_view){
            //each row is 16 pixels in height + 2 padding
            data_box_height = (18 * n_rows);
          }else{
            //each row is 16 pixels in height + 2 padding + bottom scrollbar height
            data_box_height = (18 * this.rows_per_view) + 12;
          }
          box_style = `left:${this.computed_idx_box_width + 1}px; width:${this.computed_table_width}px; height:${data_box_height}px;`;
        }
        return box_style;
      }
    },
    methods: {
      sort_column: function(col_index){
        if(col_index === this.sort_col_idx){
          this.sort_descend = !this.sort_descend;
        }else{
          this.sort_col_idx = col_index;
        }

        const self = this;
        this.sheet_rows.sort(sort_function);
        function sort_function(a, b){
          let x = a[self.sort_col_idx];
          let y = b[self.sort_col_idx];
          if(typeof x === 'string'){
            x = x.toLowerCase();
          }
          if(typeof y === 'string'){
            y = y.toLowerCase();
          }
          if(self.sort_descend){
            return ((x < y) ? 1 : ((x > y) ? -1 : 0));
          }else{
            return ((x < y) ? -1 : ((x > y) ? 1 : 0));
          }
        }
      },
      scroll_table: function(e){
        document.querySelector('#vert').scrollTop = e.currentTarget.scrollTop;
        document.querySelector('#hor').scrollLeft = e.currentTarget.scrollLeft;
      },
      cell_click: function(e,row_idx,col_idx){
        if(this.selected_td !== null){
          this.selected_td.classList.remove('worksheetComp_tdData__selected');
        }
        e.currentTarget.classList.add('worksheetComp_tdData__selected');
        this.selected_td = e.currentTarget;
        //we don't count the 'Index' column for the column index-so subtract 1
        this.$emit('work_sheet_comp_cell',{row: row_idx, col: col_idx-1});
      }
    },
    mounted(){
      if(this.css_variables !== null){
        for(let key of Object.keys(this.css_variables)){
          this.$el.style.setProperty(`--${key}`, this.css_variables[key]);
        }
      }
      this.sort_col_idx = 0;
      this.sort_descend = this.sort_descending;
    }
  }
</script>

<style lang="less">
  :root {
    --worksheet_comp_border_color: black;
    --worksheet_comp_background_color: white;

    --worksheet_comp_table_color: black;

    --worksheet_comp_row_hover_border: solid 1px black;
    --worksheet_comp_cell_selected_color: #0f9d58;
  }

  .worksheetComp {
    position: relative;
    width: 100%;

    &_idxBox {
      background-color: var(--worksheet_comp_background_color);
      position: absolute;
      left: 0;
      top: 0;
      padding: 4px 0 4px 18px;
      font-size: 14px;
      cursor: pointer;
    }

    &_table {
      background-color: var(--worksheet_comp_table_background_color);
      border-collapse: collapse;
      table-layout: fixed;
    }

    &_headingsBox {
      background-color: var(--worksheet_comp_background_color);
      position: absolute;
      top: 0;
      padding: 4px 0 4px 0;
      overflow: hidden;
    }

    &_vertBox {
      background-color: var(--worksheet_comp_background_color);
      position: absolute;
      left: 0;
      top: 29px;
      border-left: solid 2px var(--worksheet_comp_border_color);
      overflow: hidden;
    }

    &_th {
      text-align: center;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      padding: 1px;
      font-family: sans-serif;
      font-weight: bold;
      font-size: 14px;
      color: var(--worksheet_comp_table_color);
      cursor: pointer;
    }

    &_td {
      text-align: right;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      padding: 1px;
      font-family: sans-serif;
      font-size: 14px;
      color: var(--worksheet_comp_table_color);
    }

    &_dataBox {
      background-color: var(--worksheet_comp_background_color);
      position: absolute;
      top: 29px;
      border-right: solid 2px var(--worksheet_comp_border_color);
      overflow: auto;

      &::-webkit-scrollbar-track {
        border-radius: 10px;
        background-color: transparent;
      }
      &::-webkit-scrollbar {
        height: 12px;
        width: 12px;
        background-color: transparent;
      }
      &::-webkit-scrollbar-thumb {
        border-radius: 10px;
        -webkit-box-shadow: inset 0 0 8px rgba(0,0,0,.3);
        background-color: #D62929;
      }
    }

    &_trData {
      &:hover {
        border-bottom: var(--worksheet_comp_row_hover_border);
      }
    }

    &_tdData__selected {
      color: var(--worksheet_comp_cell_selected_color);
    }
  }
</style>