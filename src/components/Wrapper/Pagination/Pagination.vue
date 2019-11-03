<template>
  <div id='pagination'>
    <transition name='changePages' mode='out-in'>
      <component :is='view'></component>
    </transition>
  </div>
</template>

<script>
import {bus} from '../../../main.js'
import {axios} from '../../../main.js'

import Pages from './Pages/Pages.vue'
import SearchPages from './SearchPages/SearchPages.vue'

export default {
  components: {
    Pages: Pages,
    SearchPages: SearchPages,
  },
  data() {
    return {
      view: Pages
    };
  },
  created(){
    bus.$on('search', data => {
        if(data == true){
          this.view = SearchPages;
        }else{
          this.view = Pages;
        }
       })
  }
}
</script>


<style lang='less'>
.changePages-enter-active, .changePages-leave-active {
  transition: opacity 1s linear;
}
.changePages-enter, .changePages-leave-to
/* .component-fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
#pagination{
  background: #333;
  border: none;
}
</style>