<template>
<div id='searchPages' :class="{display: !isShow}"> <!--eslint-disable-next-line-->
<router-link :to="{ path: 'search', query: {result: value} }" @click.native = 'clickFirstPage' active-class='active' :class="{active: isOnePage}" tag='button' :key='1' exact>
1
</router-link> <!--eslint-disable-next-line-->
<router-link v-for='page in pages' :to="{ path: 'search', query: {page: page.page, result: page.result} }" @click.native = 'click(page.page)' active-class='active' tag='button' :key='page.id' exact>
{{page.page}}
</router-link>
</div>
</template>

<script>
import { bus } from '../../../../main';

export default {
  data() {
    return {
      pages: [],
      value: '',
      isShow: false,
      isOnePage: false,
    };
  },
  methods: {
    click(page) {
      let time = new Date();
      bus.$emit('searchPage', page);
      console.log('click --' + time.getTime())
    },
    clickFirstPage() {
      bus.$emit('searchFirstPage', this.pages[0].result);
    },
  },
  created() {
    bus.$emit('isCreatedSearchPages', true)
    bus.$on('searchPages', (data) => { // eslint-disable-next-line
      const tmpPages = Math.round(data / 10);
      const tmpArr = [];
      if (tmpPages > 0) { // eslint-disable-next-line
        for (let i = 2; i < tmpPages + 1; i++) {
          tmpArr.push(i);
          this.isOnePage = false;
        }
      } else {
        this.isOnePage = true;
      }
      bus.$emit('counterPages', tmpArr);
    });
    bus.$on('iShow', (data) => {
      this.isShow = data;
    });
    bus.$on('search', (data) => {
      this.isShow = !data;
    }); // eslint-disable-next-line
    bus.$on('valForPages', (data) => {
      console.log(data)
      this.value = data;
      const str = data;
      bus.$on('counterPages', (dat) => { // eslint-disable-next-line
        console.log(dat)
        this.pages = dat.map(item => {
          return {
            result: str,
            page: item,
          };
        });
      });
    });
  },
  beforeDestroy() {
    bus.$off('searchPages');
    bus.$off('valForPages');
    bus.$off('newValue');
  },
};
</script>

<style lang='less'>
.display{
display: none !important;
}
#searchPages{
    display: flex;
    flex-direction: row;
    justify-content: space-around;
    align-items: center;
    position: relative;
    margin: 0 auto;
    button{
      display: inline-block;
      position: relative;
      border: none;
      background: #808080;
      outline: none;
      text-align: center;
      color: #565656;
      font-family: 'Roboto', sans-serif;
    }
    .active,
    button:hover{
        cursor: pointer;
        color: #fff;
        text-shadow: 0 0 20px rgb(255,224,27), 
                     0 0 20px rgb(255,224,27),
                     0 0 20px rgb(255,224,27),
                     0 0 20px rgb(255,224,27),
                     0 0 20px rgb(255,224,27);
    }
  }
@media(min-width: 768px){
    #searchPages{
      width: 500px;
      height: 60px;
      button{
        width: 40px;
        height: 40px;
        border-radius: 4px;
        box-shadow: inset 0 5px 10px rgba(0,0,0,0.1), 0 2px 5px rgba(0,0,0,0.5);
        font-size: 20px;
        line-height: 40px;
      }
    }
}
@media(min-width: 550px) and (max-width: 767px){
    #searchPages{
      width: 500px;
      height: 60px;
      button{
        width: 36px;
        height: 36px;
        border-radius: 4px;
        box-shadow: inset 0 5px 10px rgba(0,0,0,0.1), 0 2px 5px rgba(0,0,0,0.5);
        font-size: 18px;
        line-height: 36px;
      }
    }
}
@media(min-width: 320px) and (max-width: 549px){
    #searchPages{
      width: 100%;
      height: 60px;
      button{
        width: 25px;
        height: 25px;
        border-radius: 4px;
        box-shadow: inset 0 5px 10px rgba(0,0,0,0.1), 0 2px 5px rgba(0,0,0,0.5);
        font-size: 18px;
        line-height: 25px;
      }
    }
}
</style>
