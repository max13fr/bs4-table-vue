# bs4-table-vue
Bootstrap4 table for Vue (with global search, filters, multi sorts, ...)

# import
    window.Vue.component('bs4-table', require('bs4-table/src/bs4-table.vue'));

# usage
    <div id="vuejs">
      <bs4-table :columns="columns" :values="users" paginated multi-sort show-columns-picker bordered hover striped>
        <template slot="header-title"><h3>Users list</h3></template>
        <template slot="toolbar-right">
          <button class="btn btn-primary" @click="add()"><i class="fas fa-user-plus"></i> Add</button>
        </template>
        <template slot="render-role_name" slot-scope="{line, content}"><a :href="'/admin/roles/'+line.role_id">{{content}}</a></template>
        <template slot="render-actions" slot-scope="{line}">
          <button class="btn btn-primary btn-xs" @click="edit(line)"><i class="fas fa-user-edit fa-fw"></i></button>
          <button class="btn btn-primary btn-xs" @click="delete(line)"><i class="fas fa-user-times fa-fw"></i></button>
        </template>
      </bs4-table>
    </div>
      
    <script>
      var vuejs = new Vue({
          el: '#vuejs',
          data: {
              users: [
                  { username: "jdoe", firstname: "john", lastname: "doe", role_id: 10, role_name: "Employee", title: 'Developer'},
                  { username: "umartin", firstname: "Ugo", lastname: "Martin", role_id: 10, role_name: "Employee", title: 'Developer', mail: 'ugo.martin@company.com' },
                  { username: "abob", firstname: "Alice", lastname: "Bob", role_id: 1, role_name: "Admin", title: 'Boss'},
              ],
              columns: [
                  { key: 'username',         title: 'Username' },
                  { key: 'firstname',        title: 'Firstname' },
                  { key: 'lastname',         title: 'Lastname' },
                  { key: 'role_name',        title: 'Role' },
                  { key: 'mail',             title: 'Email' },
                  { key: 'phone',            title: 'Phone', defaultHidden: true },
                  { key: 'cellphone',        title: 'Cellphone', defaultHidden: true },
                  { key: 'title',            title: 'Title' },
                  { key: 'zone_name',        title: 'Office' },
                  { key: 'actions',          title: 'Actions', sortable: false, filterable: false, cellClass: 'text-center', contentFunction: function() { return ''; } },
              ],
          },
          methods: {
              add: function() {
                  alert('add new user');
              },
              edit: function(user) {
                  alert('edit user ' + user.username);
              },
              delete: function(user) {
                  alert('delete user ' + user.username);
              },
          },
      });
    </script>
