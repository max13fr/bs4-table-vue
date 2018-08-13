<template>
  <div class="bs4-table">
    <div class="row">
      <div class="col-sm-12 form-inline">
        <div class="d-flex w-100" style="padding: 10px 2px 10px 2px;">
          <div class="flex-grow-1 align-self-center"><slot name="header-title"></slot></div>
          <div class="align-self-center mr-3" v-if="this.$slots['toolbar-left']"><slot name="toolbar-left"></slot></div>
          <div v-if="globalFilterable" class="input-group align-self-center">
            <input type="search" v-model="globalFilter" placeholder="Rechercher..." size="50" class="form-control">
            <div class="input-group-append">
              <span class="input-group-text">
                <i class="fas fa-search"></i>
              </span>
            </div>
          </div>
          <div v-if="showColumnsPicker" class="btn-group align-self-center ml-3">
            <button class="btn btn-secondary dropdown-toggle" data-toggle="dropdown">Colonnes</button>
            <div class="dropdown-menu dropdown-menu-right" style="padding: 0px !important;">
              <div class="list-group">
                <a v-for="column in curColumns" @click.prevent.stop="toggleColumn(column)" href="#" class="list-group-item list-group-item-action" style="white-space: nowrap;">
                  <i :class="['fas', (column.visible ? 'fa-check-square' : 'fa-square'), 'fa-fw']"></i> {{column.title}}
                </a>
              </div>
            </div>
          </div>
          <div class="align-self-center ml-3" v-if="this.$slots['toolbar-right']"><slot name="toolbar-right"></slot></div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-sm-12">
        <div id="loadingdiv" :class="{'bs4-table-loading': this.loading , 'bs4-table-loading-hidden': !this.loading}">
          <div class="spinner"></div>
        </div>
        <table :class="{'bs4-table-table': true, 'table': true, 'table-bordered': bordered, 'table-hover': hover, 'table-sm': sm, 'table-striped': striped}">
          <thead>
            <tr>
              <th v-for="column in curColumnsVisible" @click="clickOnColumn($event, column)" track-by="column" :class="[{'sortable': sortable && column.sortable}, column.headerClass]">
                <div class="float-right">
                  <i v-if="columnsFilterable && column.columnFilterable" class="fas fa-filter fa-fw mr-1" style="cursor:pointer;" rel="filter" title="Filtrer sur cette colonne"></i>
                  <i v-if="sortable && column.sortable" :class="['fas', columnSortClass(column.key)]"></i>
                </div>
                {{column.title}}
                <template v-if="columnsFilterable && column.columnFilterable && column.filter.enable">
                  <br />
                  <template v-if="globalFilterable && globalFilter !== ''"><span class="text-muted font-weight-normal font-italic">Filtrage global en cours</span></template>
                  <template v-else><input v-model="column.filter.query" type="search" class="form-control" rel="input-filter" placeholder="Filtre..." @keyup="curPage = 1"/></template>
                </template>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="line in filteredValuesSortedPaginated" track-by="line" @click="rowClickHandler($event, line)">
              <td v-for="column in curColumnsVisible" track-by="column" :class="column.cellClass">
                <slot :name="'render-' + column['key']" :column="column" :line="line" :content="column.contentFunction(line, column)">{{column.contentFunction(line, column)}}</slot>
              </td>
            </tr>
            <tr v-if="filteredValuesSorted.length === 0">
              <td :colspan="curColumnsVisible.length" class="text-center">Aucune ligne trouvée</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div class="row">
      <div v-if="paginated" class="col-sm-12">
        <ul class="pagination float-right">
          <li class="page-item">
            <button class="page-link" aria-label="Previous" @click="curPage -= 1" :disabled="curPage <= 1">&laquo;</button>
          </li>
          <li v-for="index in displayPages" class="page-item" :class="{active: curPage === index}">
            <button v-if="index === 0" class="page-link" :disabled="true">...</button>
            <button v-else class="page-link" @click="curPage = index" :disabled="curPage === index">{{index}}</button>
          </li>
          <li class="page-item">
            <button class="page-link" aria-label="Next" @click="curPage += 1" :disabled="curPage >= nbPages">&raquo;</button>
          </li>
        </ul>
        <div class="form-inline">
          <strong>Total</strong>: {{filteredValues.length}} lignes &nbsp;
          <select v-model="curPageSize" class="custom-select">
            <option v-for="i in [10, 20, 50, 100, -1]" :value="i">
              <template v-if="i !== -1">{{i}} / page </template>
              <template v-else>Tout afficher</template>
            </option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
    .bs4-table-table > thead > tr > th {
        vertical-align: top;
    }
    .bs4-table-table > thead > tr > th.sortable {
        cursor: pointer;
    }
    /* add cross for filter input on chrome*/
    .bs4-table input[type="search"] {
        -webkit-appearance: searchfield;
    }
    .bs4-table input[type="search"]::-webkit-search-cancel-button {
        -webkit-appearance: searchfield-cancel-button;
    }

    .bs4-table-loading .spinner {
        border: 16px solid #f3f3f3; /* Light grey */
        border-top: 16px solid #3498db; /* Blue */
        border-radius: 50%;
        width: 120px;
        height: 120px;
        animation: spin 2s linear infinite;
        position: absolute;
        left: 50%;
        top: 50%;
        margin: -60px 0 0 -60px;
    }

    @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
    }

    .bs4-table-loading{
        position: absolute;
        z-index: 99;
        background-color: #ddd;
        opacity: 0.5;
        width: 100%;
        height: 100%;
    }

    .bs4-table-loading-hidden {
        display: none;
    }
</style>

<script>
var _ = require('lodash');

module.exports = {
    name: 'bs4-table',
    props: {
        /**
         * The column titles, required
         */
        columns: {
            type: Array,
            required: true,
        },
        /**
         * The rows, an Array of objects
         */
        values: {
            type: Array,
            required: false,
        },
        /**
         * Enable/disable global table sorting, optional, default true
         */
        sortable: {
            type: Boolean,
            required: false,
            default: true,
        },
        /**
         * Enable/disable multicolumn sorting, optional, default true.
         * Also sortable must be enabled for this function to work.
         */
        multiSort: {
            type: Boolean,
            required: false,
            default: true,
        },
        /**
         * Setting default sort order. Ex: ['firstname', {'lastname' => 'desc'}, ..]
         */
        defaultSort: {
            type: String,
            required: false,
            default: null,
        },
        /**
         * Enable/disable global filter, optional, default true
         */
        globalFilterable: {
            type: Boolean,
            required: false,
            default: true,
        },
        /**
         * Enable/disable filters on columns, optional, default true
         */
        columnsFilterable: {
            type: Boolean,
            required: false,
            default: true,
        },
        /**
         * Define if filters are case sensitive, default false
         */
        filterCaseSensitive: {
            type: Boolean,
            required: false,
            default: false,
        },
        /**
         * Enable/disable column picker to show/hide table columns, optional, default false
         */
        showColumnsPicker: {
            type: Boolean,
            required: false,
            default: false,
        },
        /**
         * Enable/disable pagination for the table, optional, default false
         */
        paginated: {
            type: Boolean,
            required: false,
            default: false,
        },
        /**
         * If pagination is enabled defining the page size, optional, default 10
         */
        pageSize: {
            type: Number,
            required: false,
            default: 10,
        },
        /**
         * Enable/disable table-bordered class on table
         */
        bordered: {
            type: Boolean,
            required: false,
            default: false,
        },
        /**
         * Enable/disable table-hover class on table
         */
        hover: {
            type: Boolean,
            required: false,
            default: false,
        },
        /**
         * Enable/disable table-sm class on table
         */
        sm: {
            type: Boolean,
            required: false,
            default: false,
        },
        /**
         * Enable/disable table-striped class on table
         */
        striped: {
            type: Boolean,
            required: false,
            default: false,
        },
        /**
         * Function to handle clicks on row
         */
        rowClickHandler: {
            type: Function,
            required: false,
            default: function () {}
        },
    },
    data: function () {
        return {
            curColumns:   [],
            curSort:      [], // format: [ {'column': 'key1', 'order': 'asc'}, {'column': 'key2', 'order': 'desc'}, ... ]
            curPageSize:  10,
            curPage:      1,
            globalFilter: '',
            loading: false,
        };
    },
    /**
     * Once mounted and ready to start
     */
    mounted: function () {
        var self = this;

        // parsing default sort
        if (self.defaultSort) {
            if (Array.isArray(self.defaultSort)) {
                self.default.forEach(function(val) {
                    if (typeof(val) === 'string') {
                        self.curSort.push({'column': val, 'order': 'asc'});
                    }
                    else if(typeof(val) === 'object' && Object.keys(val).length === 1) {
                        var col = Object.keys(val)[0];
                        var order = val[col].toLowerCase();
                        if (order === 'asc' || order === 'desc') {
                            self.curSort.push({'column': col, 'order': order});
                        }
                        else {
                            console.error('[bs4-table] invalid defaultSort parameter: unknown order "' + order + '" (waiting for "asc" or "desc")');
                        }
                    }
                    else {
                        console.error('[bs4-table] invalid defaultSort parameter: wait for an array of strings and/or objects');
                    }
                });
            }
            else {
                console.error('[bs4-table] invalid defaultSort parameter: wait for an array');
            }
        }

        // set page size from user
        self.curPageSize = self.pageSize;

        // build current columns
        self.columns.forEach(function (column) {
            self.curColumns.push(self.buildColumnObject(column));
        });
    },
    /**
     * On created register on CellDataModified event
     */
    created: function () {
        var self = this ;
        this.$on('cellDataModifiedEvent', self.fireCellDataModifiedEvent);
    },
    /**
     * On destroy unregister the event
     */
    beforeDestroy: function(){
        var self = this ;
        this.$off('cellDataModifiedEvent', self.fireCellDataModifiedEvent);
    },
    watch: {
        // update curColumns from columns parameter
        columns: function () {
            var self = this;
            self.curColumns = [];
            self.columns.forEach(function (column) {
                self.curColumns.push(self.buildColumnObject(column));
            });
        },
    },
    computed: {
        // return only visible columns
        curColumnsVisible: function () {
            return _.filter(this.curColumns, 'visible');
        },
        // get filters values
        filteredValues: function() {
            var self = this;

            // global filter
            if (self.globalFilterable && self.globalFilter !== '') {
                var globalFilter = self.globalFilter;
                if (!self.filterCaseSensitive) {
                    globalFilter = globalFilter.toLowerCase();
                }

                return self.values.filter(function(line) {
                    // default to hidden
                    var displayed = false;

                    // try to find value in one column
                    self.$lodash.forEach(self.curColumnsVisible, function(column) {
                        var content = String(column.contentFunction(line, column));
                        if (!self.filterCaseSensitive) {
                            content = content.toLowerCase();
                        }
                        if (content.indexOf(globalFilter) !== -1) {
                            displayed = true;
                            return false; // end column loop
                        }
                    });

                    return displayed;
                });
            }

            // columns filters
            if (self.columnsFilterable) {
                var listColumns = [];
                self.curColumnsVisible.forEach(function(column) {
                    if (column.columnFilterable && column.filter.enable && column.filter.query !== '') {
                        var col = _.cloneDeep(column);
                        if (!self.filterCaseSensitive) {
                            col.filter.query = col.filter.query.toLowerCase();
                        }
                        listColumns.push(col);
                    }
                });

                if (listColumns.length) {
                    return self.values.filter(function(line) {
                        var displayed = true;
                        self.$lodash.forEach(listColumns, function(column) {
                            var content = String(column.contentFunction(line, column));
                            if (!self.filterCaseSensitive) {
                                content = content.toLowerCase();
                            }
                            if (content.indexOf(column.filter.query) === -1) {
                                displayed = false;
                                return false; // end column loop
                            }
                        });
                        return displayed;
                    });
                }
            }

            // no filter
            return self.values;
        },
        // get filtered values sorted
        filteredValuesSorted: function () {
            var self = this;
            if (self.curSort.length === 0) {
                return self.filteredValues;
            }

            // preparer sorters
            var listSorters = [];
            self.curSort.forEach(function(info) {
                var column = _.find(self.curColumns, {'key': info['column']});
                listSorters.push(function(line1, line2) {
                    var factor = (info['order'] === 'asc' ? 1 : -1);
                    return column.sorterFunction(column.contentFunction(line1, column), column.contentFunction(line2, column)) * factor;
                });
            });
            
            // sort
            return self.filteredValues.sort(function(line1, line2) {
                var result = 0;
                for(var i = 0; result === 0 && i < listSorters.length; i++) {
                    result = listSorters[i](line1, line2);
                }
                return result;
            });
        },
        // get filtereds values sorted for the current page (return all values if no pagination)
        filteredValuesSortedPaginated: function() {
            if (!this.paginated || this.curPageSize === -1) {
                return this.filteredValuesSorted;
            }

            var startIndex = (this.curPage - 1) * this.curPageSize;
            var endIndex = Math.min(startIndex + this.curPageSize, this.filteredValuesSorted.length + 1);
            return this.filteredValuesSorted.slice(startIndex, endIndex);
        },
        // compute nb pages
        nbPages: function () {
            if (this.curPageSize === -1 || this.filteredValues.length === 0) {
                return 1;
            }
            return Math.ceil(this.filteredValues.length / this.curPageSize);
        },
        // get display pages
        displayPages: function() {
            if (this.nbPages <= 9) {
                return _.range(1, this.nbPages + 1);
            }

            var i = this.curPage;
            var n = this.nbPages;

            if (this.curPage <= 5) {
                return [1, 2, 3, 4, 5, 6, 7, 0, n]; // 0 represents '···'
            }
            if (this.curPage >= this.nbPages - 4) {
                return [1, 0, n-6, n-5, n-4, n-3, n-2, n-1, n]; // 0 represents '···'
            }
            return [1, 0, i-2, i-1, i, i+1, i+2, 0, n]; // 0 represents '···'
        },
    },
    methods: {
        clickOnColumn(evt, column) {
            var $target = $(evt.target);
            if ($target.attr('rel') === 'filter') {
                this.$set(column.filter, 'enable', !column.filter.enable);
                // reset page if enable/disable a filter
                if (column.filter.query !== '') {
                    this.curPage = 1;
                }
            }
            else if ($target.attr('rel') !== 'input-filter') {
                return this.sortBy(evt, column);
            }
        },
        sortBy: function (evt, column) {
            var self = this;
            var key = column.key;

            if (!self.sortable || !column.sortable) {
                return;
            }

            // get current sort status for the column
            var curColumnSortIndex = _.findIndex(self.curSort, {'column': key});
            var newOrder = 'asc';
            if (curColumnSortIndex !== -1) {
                newOrder = (self.curSort[curColumnSortIndex]['order'] === 'asc' ? 'desc' : undefined);
            }

            // prepare new sort (to avoid multiple refresh of computed sorted values)
            var newSort = [];

            // keep cur sort if multicolumn
            if (self.multiSort && evt.shiftKey) {
                newSort = _.cloneDeep(self.curSort);
                if (curColumnSortIndex === -1) {
                    newSort.push({'column': key, 'order': newOrder});
                }
                else if (newOrder !== undefined) {
                    newSort[curColumnSortIndex]['order'] = newOrder;
                }
                else {
                    newSort.splice(curColumnSortIndex, 1);
                }
            }
            else if(newOrder !== undefined) { // new sort
                newSort.push({'column': key, 'order': newOrder});
            }

            // update current sort
            self.$set(self, 'curSort', newSort);
        },
        columnSortClass: function(key) {
            var colSort = _.find(this.curSort, {'column': key});
            var icon = 'fa-sort';
            if (colSort !== undefined) {
                icon += (colSort.order === 'asc' ? '-up' : '-down');
            }
            return icon;
        },
        toggleColumn: function (column) {
            column.visible = !column.visible;
        },
        // build internal column object from user column entry
        buildColumnObject: function (column) {
            var obj = {
                key: column.key,
                title: column.key,
                visible: true,
                sortable: true,
                globalFilterable: true,
                columnFilterable: true,
                filter: {
                    enable: false,
                    query: ''
                },
                headerClass: '',
                cellClass: '',
                contentFunction: function(line, column) {
                    return line[column['key']];
                },
                sorterFunction: function(content1, content2) {
                    return String(content1).localeCompare(String(content2));
                },
            };

            if (typeof(column.title) === 'string') {
                obj.title = column.title;
            }
            if (typeof(column.defaultHidden) !== "undefined") {
                obj.visible = ! Boolean(column.defaultHidden);
            }
            if (typeof(column.sortable) !== "undefined") {
                obj.sortable = Boolean(column.sortable);
            }
            if (typeof(column.globalFilterable) !== "undefined") {
                obj.globalFilterable = Boolean(column.globalFilterable);
            }
            if (typeof(column.columnFilterable) !== "undefined") {
                obj.columnFilterable = Boolean(column.columnFilterable);
            }
            if (typeof(column.defaultFilter) === "string") {
                obj.filter.enable = true;
                obj.filter.query = column.defaultFilter;
            }
            if (['string', 'object'].indexOf(typeof(column.headerClass)) !== -1) {
                obj.headerClass = column.headerClass;
            }
            if (['string', 'object'].indexOf(typeof(column.cellClass)) !== -1) {
                obj.cellClass = column.cellClass;
            }
            if (typeof(column.contentFunction) === 'function') {
                obj.contentFunction = column.contentFunction;
            }

            return obj;
        },
    },
}
</script>
