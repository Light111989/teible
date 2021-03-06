<template>
  <div class="datatable">
    <div class="datatable__wrapper">
      <div class="datatable__heading">
        <data-table-filter :filter.sync="options.filter" class="datatable__unit"/>
        <div class="datatable__unit datatable__text">
          <span v-if="total">
            Showing <span v-text="from === to && to === this.total ? 'the last entry' : from + ' to ' + to"/> of {{ total }} records
          </span>
          <span v-else>No records</span>
        </div>
      </div>
      <table class="datatable__content" cellspacing="0" cellpadding="0">
        <data-table-head :columns="columns" :sort-by.sync="options.sortBy" :sort-desc.sync="options.sortDesc"/>
        <data-table-body :columns="columns" :items="actualItems"/>
      </table>
      <data-table-pagination :per-page="perPage" :page.sync="page" :total="total"/>
    </div>
  </div>
</template>
<script>
import { load, defaultProps } from './helpers'
import DataTableBody from './DataTableBody.vue'
import DataTableHead from './DataTableHead.vue'
import DataTablePagination from './DataTablePagination.vue'
import DataTableFilter from './DataTableFilter.vue'

export default {
  name: 'DataTable',
  components: { DataTableBody, DataTableHead, DataTablePagination, DataTableFilter },
  props: {
    items: {
      type: [Array, Function],
      required: true
    },
    perPage: {
      type: Number,
      default: 10
    },
    sortBy: {
      type: String,
      default: ''
    },
    sortDesc: {
      type: Boolean,
      default: false
    },
    filter: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      actualItems: [],
      vnodes: [],
      total: 0,
      page: 1,
      options: {
        sortBy: this.sortBy,
        sortDesc: this.sortDesc,
        filter: this.filter
      }
    }
  },
  computed: {
    identifier () {
      return `by:${this.sorting.by}|order:${this.sorting.order}|filter:${this.options.filter}|page:${this.page}|per_page:${this.perPage}`
    },
    asynchronous () {
      return this.items instanceof Function
    },
    columns () {
      return this.vnodes.map(vnode => {
        let { componentOptions: { Ctor: { options: { props } }, propsData, children }, data: { scopedSlots, attrs, class: dynamicClass, staticClass } } = vnode
        let { field, label, sortable, filterable } = defaultProps(props, propsData)
        return {
          field,
          label,
          sortable,
          filterable,
          scopedSlots,
          children,
          attrs,
          dynamicClass,
          staticClass
        }
      })
    },
    filterable () {
      return this.columns
        .filter(column => {
          return column.filterable
        })
        .map(column => {
          return column.field
        })
        .filter(field => field)
    },
    filtering () {
      return {
        query: this.options.filter.toLowerCase(),
        fields: this.filterable
      }
    },
    paging () {
      return {
        page: this.page,
        perPage: this.perPage
      }
    },
    sorting () {
      return {
        by: this.options.sortBy,
        order: !this.options.sortDesc ? 'asc' : 'desc'
      }
    },
    from () {
      return (this.page - 1) * this.perPage + 1
    },
    to () {
      let x = this.page * this.perPage
      return this.total < x ? this.total : x
    }
  },
  watch: {
    items: 'loadItems',
    identifier: {
      immediate: true,
      handler: 'loadItems'
    },
    sortBy: {
      immediate: true,
      handler (val) {
        this.$set(this.options, 'sortBy', val)
      }
    },
    sortDesc: {
      immediate: true,
      handler (val) {
        this.$set(this.options, 'sortDesc', val)
      }
    },
    filter: {
      immediate: true,
      handler (val) {
        this.$set(this.options, 'filter', val)
      }
    },
    'options.sortBy' (val) {
      this.$emit('update:sortBy', val)
    },
    'options.sortDesc' (val) {
      this.$emit('update:sortDesc', val)
    },
    'options.filter' (val) {
      this.$emit('update:filter', val)
    }
  },
  created () {
    this.loadSlots()
  },
  methods: {
    loaded (data) {
      this.$emit('loaded', data)
      this.actualItems = data.items
      this.total = data.total
    },
    loadSlots () {
      // $slots is not reactive
      this.vnodes = !this.$slots.default ? [] : this.$slots.default.filter(vnode => vnode.componentOptions)
    },
    loadItems () {
      this.load(this.items, this.filtering, this.sorting, this.paging)
    },
    async load (items, filtering, sorting, paging) {
      if (this.asynchronous) {
        this.loaded(await items(filtering, sorting, paging))
        return
      }

      this.loaded(load(items, filtering, sorting, paging))
    }
  }
}
</script>
<style lang="scss">
@import "~@hiendv/bem-sass";
*, *::after, *::before {
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}

.datatable {
  color: $color;
  font: 1rem/1.5 -apple-system,BlinkMacSystemFont,Roboto,Helvetica,Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol";
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;

  @include e('wrapper') {
    position: relative;
    display: block;
    text-align: left;
    width: 100%;
  }

  @include e('heading') {
    margin-bottom: 1em;
    display: table;
    table-layout: fixed;
    width: 100%;
  }

  @include e('unit') {
    width: 50%;
    display: table-cell;
  }

  @include e('text') {
    padding-left: 1em;
  }

  @include e('content') {
    min-width: 100%;
    border: solid 1px $border-color;
    table-layout: fixed;
  }
}
</style>
