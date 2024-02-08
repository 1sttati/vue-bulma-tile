# vue-bulma-table
Vue DataTile with Bulma style

## Getting Started
### Installing
```
npm install vue-bulma-tile --save
```
or
```
yarn add vue-bulma-tile
```

### Setup
```js
import DataTile from 'vue-bulma-tile'

Vue.component('data-tile', DataTile)
```

## Example:

### html
```html
<data-table
  :data="data"                                    # required - array of data
  :filterableFields="fields"                      # required - set columns
>

  <template v-slot="{ data }">
    <div class="box mb-3">
      {{ data.firstname }}
    </div>
  </template>
  
</data-table>
```

### script
```js
export default {
  data () {
    return {
      fields: [ 'firstname', 'lastname', 'age' ]
    }
  }
}
```
