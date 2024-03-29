# vue-bulma-tile
Vue Data Tile with Bulma style

## Getting Started
### Installing
```
npm install https://github.com/1sttati/vue-bulma-tile.git
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
<data-tile
  :data="data"                                    # required - array of data
  :filterableFields="fields"                      # required - set columns
>

  <template v-slot="{ data }">
    <div class="box mb-3">
      {{ data.firstname }}
    </div>
  </template>
  
</data-tile>
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
