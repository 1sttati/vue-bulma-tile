<template>
  <div style="position: relative;">
    <nav v-if="lengthChange || filterable" class="level is-mobile">
      <div v-if="lengthChange && !scrollable" class="level-left">
        <div class="level-item">
          <div class="select" :class="inputClass">
            <select v-model="perPage">
              <option v-for="(length, i) in pageLength" :value="length" :key="i">{{ length }}</option>
            </select>
          </div>
        </div>
      </div>
      <div v-if="filterable" class="level-right">
        <div class="level-item">
          <div class="field">
            <p class="control has-icons-left has-icons-right">
              <input v-model="tableFilter" class="input" :class="inputClass" type="text" placeholder="search">
              <span class="icon is-small is-left">
                <i class="fa fa-search" />
              </span>
            </p>
          </div>
        </div>
      </div>
    </nav>
    <div class="tile ancestor mb-5">
      <div v-for="(d, index) in dataSet" :key="index" class="tile">
        <slot :data="d">
          <!--  -->
        </slot>
      </div>
    </div>
    <nav v-if="pagination && !scrollable" class="pagination is-centered" :class="inputClass" role="navigation" aria-label="pagination">
      <a class="pagination-previous" :disabled="from - perPage < 0" @click="previousPage">Previous</a>
      <a class="pagination-next" :disabled="from + perPage >= filteredData.length" @click="nextPage">Next page</a>
      <ul class="pagination-list">
        <li><a v-if="currentPage > 1" class="pagination-link" @click="toFirstPage">1</a></li>
        <li><span v-if="currentPage - 2 > 1" class="pagination-ellipsis">&hellip;</span></li>
        <li>
          <a v-if="currentPage - 1 > 1" class="pagination-link" @click="previousPage">{{ currentPage - 1 }}</a>
        </li>
        <li><a class="pagination-link is-current">{{ currentPage }}</a></li>
        <li>
          <a v-if="currentPage + 1 < lastPage" class="pagination-link" @click="nextPage">{{ currentPage + 1 }}</a>
        </li>
        <li><span v-if="currentPage + 2 < lastPage" class="pagination-ellipsis">&hellip;</span></li>
        <li><a v-if="currentPage < lastPage" class="pagination-link" @click="toLastPage">{{ lastPage }}</a></li>
      </ul>
    </nav>
    <div v-if="loading" style="display: flex; align-items: center; position: absolute; top: 0; width: 100%; height: 100%; background-color: rgba(0,0,0,0.7); box-shadow: 0 0 20px 5px rgba(0,0,0,0.7);">
      <div style="margin: auto; color: white;">
        <i class="fa fa-circle-notch fa-3x fa-spin" />
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  props: {
    filterableFields: {
      type: Array,
      default () {
        return []
      }
    },
    data: {
      type: Array,
      required: true
    },
    pageLength: {
      type: [Array, Number],
      default () {
        return [10, 25, 50]
      }
    },
    loading: {
      type: Boolean,
      default: false
    },
    lengthChange: {
      type: Boolean,
      default: true
    },
    filterable: {
      type: Boolean,
      default: true
    },
    pagination: {
      type: Boolean,
      default: true
    },
    inputClass: {
      type: String,
      default: ''
    },
    scrollable: {
      type: Boolean,
      default: false
    },
    bodyHeight: {
      type: String,
      default: '300px'
    }
  },

  data () {
    return {
      perPage: 10,
      from: 0,
      tableFilter: '',
      columnsFilter: {},
      sort1: { field: '', order: '' },
      sort2: { field: '', order: '' },
      sort3: { field: '', order: '' },

      regexSearch: false
    }
  },

  computed: {
    columnslength () {
      return Object.keys(this.columnsFilter).length === 0
    },
    filteredData () {
      return _.filter(this.data, data => {
        for (const i in this.filterableFields) {
          if (this.regexSearch) {
            return String(this.getObjectData(data, this.filterableFields[i])).search(this.tableFilter) >= 0
          } else if (String(this.getObjectData(data, this.filterableFields[i])).toLowerCase().includes(this.tableFilter.toLowerCase())) {
            return true
          }
        }
      })
    },
    dataSort () {
      return _.orderBy(this.filteredData, [
        this.sort1.field, this.sort2.field, this.sort3.field
      ], [this.sort1.order, this.sort2.order, this.sort3.order])
    },
    dataSet () {
      if (this.scrollable) return this.dataSort
      return this.dataSort.slice(this.from, this.from + this.perPage)
    },
    lastPage () {
      return Math.ceil(this.filteredData.length / this.perPage)
    },
    currentPage () {
      return Math.ceil(this.from / this.perPage) + 1
    },
    floatHead () {
      let className = 'cursorPointer'
      if (this.scrollable) className += ' floatHead'
      return className
    }
  },

  watch: {
    dataSet (data) {
      this.$emit('currentData', data)
    },
    pageLength (data) {
      this.perPage = Array.isArray(data) ? data[0] : data
    },
    tableFilter () {
      this.from = 0
    },
    columnsFilter: {
      handler () {
        this.from = 0
      },
      deep: true
    }
  },

  beforeMount () {
    if (this.scrollable) {
      this.perPage = 1000000
    } else {
      this.perPage = Array.isArray(this.pageLength) ? this.pageLength[0] : this.pageLength
    }
  },

  methods: {
    previousPage () {
      if (this.from - this.perPage >= 0) {
        this.from = this.from - this.perPage
      }
    },
    nextPage () {
      if (this.from + this.perPage < this.filteredData.length) {
        this.from = this.from + this.perPage
      }
    },
    toFirstPage () {
      this.from = 0
    },
    toLastPage () {
      this.from = Math.floor((this.filteredData.length - 1) / this.perPage) * this.perPage
    },
    sortField (field) {
      if (this.sort1.field === field.name) {
        this.sort1.order = this.sort1.order !== 'asc' ? 'asc' : 'desc'
      } else {
        Object.assign(this.sort3, this.sort2)
        Object.assign(this.sort2, this.sort1)
        this.sort1 = {
          field: field.name,
          order: 'asc'
        }
      }
    },
    getObjectData (data, field) {
      let o = data
      field.split('.').forEach(key => {
        o = o[key] ? o[key] : ''
      })
      return o
    }
  }
}
</script>
