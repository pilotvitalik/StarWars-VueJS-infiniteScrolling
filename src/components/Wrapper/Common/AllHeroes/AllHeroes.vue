<template>
	<div id='AllHeroes'>
		  <Search></Search>
    <transition name='slide-fade' type='transition'>
		<nav class='left'  v-if='isAnimation'>
      <transition name='changeComponent' mode='out-in'>
        <component :is='view'></component>
      </transition>
		</nav>
    </transition>
    <LoadingPage v-if='showLoadPage'></LoadingPage>
	</div>
</template>

<script>
import {bus} from '../../../../main.js'

  import LoadingPage from '../../Preloader/LoadingPage/LoadingPage.vue'
  import ListCarts from './ListCarts/ListCarts.vue'
  import Search from './Search/Search.vue';
  import SearchCarts from './SearchCarts/SearchCarts.vue'

export default {
  components: {
    LoadingPage:LoadingPage,
    ListCarts: ListCarts,
    Search: Search,
    SearchCarts: SearchCarts,
  },
  data() {
    return {
	         peoples:[],
            comm: [],
            species: [],
            show: false,
            person:[],
            id: '',
            request: {
              people: false,
            },
            showLoadPage: false,
            isAnimation: true,
            isNavigation: false,
            view: ListCarts,
            isCreateSearch: false,
      };
  },
    methods: {
      descript(people) {
        let obj = {};
        let arr = [];
        if(this.show == false){
          bus.$emit('show', true);
          bus.$emit('blur', 'blur(5px)');
          document.querySelector('body').style.overflowY = 'hidden';
          document.querySelector('body').style.width = '100%';
          obj = {
            name: people.name,
            image: people.img,
            birth: people.birth_year,
            specie: people.specie,
            gender: people.gender,
            home: people.homeworld,
            film: '',
            planet: '',
          }
          this.$http.get(people.homeworld).then(responsive => {
            if(responsive.ok == true){
              bus.$emit('planet', responsive.data.name)
            }
          })
          for(let i = 0; i < people.films.length; i++){
            this.$http.get(people.films[i]).then(responsive => {
              bus.$emit('sh', false)
              if(responsive.ok == true){
                arr.push(responsive.data.title)
                bus.$emit('films', arr)
              }
            })
          }
          this.person.push(obj)
          bus.$emit('add', this.person)
        } 
      },
    },
    computed: {
      filterHeroes: function(){
        return this.peoples.filter(hero => {
          return hero.name.match(this.search);
        })
      }
    },
    created(){
      document.querySelector('body').style.overflowY = 'hidden';
      document.querySelector('body').style.pointerEvents = 'none';
      bus.$on('modalWindow', data => {
        this.descript(data);
        console.log(data)
      })
      bus.$on('modalWindo', data => {
        this.descript(data);
      })
      
      
    //----------Set width when modal window closed
      bus.$on('showCom', data => {
        this.show = data;
        document.querySelector('body').style.overflowY = 'auto';
        let withScroll = document.documentElement.clientWidth
          let outScroll = window.innerWidth
          let body = document.querySelector('body')
          if(withScroll < outScroll){
            let delta = (outScroll - withScroll)*100/outScroll;
            let newDelta = parseFloat(delta.toFixed(16), 10);
            body.style.width = (100+newDelta)+'%';
            body.style.overflowX = 'hidden';
          };
          this.person = [];
        bus.$emit('show', false);
      })
    //------------End set width-------------------------- 
    //-------------Disappear ListCarts, create SearchCarts and start animation loading
       bus.$on('search', data => {
        if(data == true){
          this.view = SearchCarts;
          this.showLoadPage = data;
        }else{
          this.view = ListCarts;
          this.showLoadPage = data;
        }
       })
       let str = '';
       bus.$on('value', data => {
        str = data;
       });
       bus.$on('created', data => {
        this.$nextTick(function() {
          this.isCreateSearch = data;
          if(this.isCreateSearch == true){
            bus.$emit('val', str)
          }
        })
       })
     //-------------------------------------------------------------   
     //---------------Stop animation loading from ListCarts to SearchCarts---------
      bus.$on('iShow', data => {
        this.showLoadPage = !data;
      })
     //----------------------------------------------------------------------------
     //-----------------Start animation loading page with InfiniteScrolling--------
     bus.$on('animationLoadingPage', data => {
      if(this.isCreateSearch == true){
        this.showLoadPage = false
      }else{
        this.showLoadPage = true
      }
      
     })
     //-----------------------------------------------------------------------------
     //--------------Stop animation loading page with InfiniteScrolling------------
     bus.$on('lengthSpecies', data => {
      bus.$on('counterStatus', counter => {
        if(data == counter){
          setTimeout(() => {
            this.showLoadPage = false
          }, 1000)
        }
      })
     })
     //-----------------------------------------------------------------------------
    },
};
</script>


<style lang='less'>

.changeComponent-enter-active, .changeComponent-leave-active {
  transition: opacity 1s linear;
}
.changeComponent-enter, .changeComponent-leave-to
/* .component-fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

.slide-fade-enter-active {
  transition: all 3s ease;
}
.slide-fade-enter{
  transform: translateY(100px);
}
.notDisplay{
  opacity: 0;
}
.view{
  animation: displayCarts 3s linear forwards;
}
@keyframes displayCarts{
  0%{
    opacity: 0;
  }
  100%{
    opacity: 1;
  }
}

#AllHeroes{
	display: flex;
	flex-direction: column;
	position: relative;
	height: auto;
	background: #333;
	.search{
		display: block;
		position: relative;
		input{
			display: block;
			position: relative;
			width: 100%;
			height: 100%;
			color: #808080;
			font-family: 'Roboto', sans-serif;
			font-weight: 500;
			background: #333;
			border: none;
			outline: none;
		}
    .title{
      display: inline-block;
      position: absolute;
      width: 100%;
      color: #808080;
      font-family: 'Roboto', sans-serif;
      pointer-events: none;
    }
		img{
			display: block;
			position: absolute;
		}
    .fa{
      display: inline-block;
      position: absolute;
      color: #808080;
    }
    .fa:hover{
      cursor: pointer;
      color: #fff;
    }
	}
	nav{
		display: block;
		position: relative;
		height: auto;
		margin: 0 auto;
	}
}
@media(min-width: 768px){
	#AllHeroes{
		padding-bottom: 168-32px;
		width: 100%;
		height: auto;
		.search{
			width: 800*100/1440%;
			height: 29px;
			margin: 107px auto 80px auto;
			input{
				padding-left: 5px;
				font-size: 18px;
				line-height: 21px;
				border-bottom: 1px solid #808080;
			}
      .title{
      bottom: 5px;
      left: 5px;
      font-size: 18px;
      line-height: 21px;
      }
			img{
				top: 2px;
				right: 5px;
			}
      .fa{
        top: 5px;
        right: 5px;
      }
		}
		nav{
			width: 1216*100/1440%;
			height: auto;
		}
	}
}
@media(min-width: 320px) and (max-width: 767px){
	#AllHeroes{
		padding-bottom: 120-24px;
		width: 100%;
		.search{
			width: 272*100/320%;
			height: 29px;
			margin: 45px auto 44px auto;
			input{
				padding-left: 5px;
				font-size: 16px;
				line-height: 19px;
				border-bottom: 1px solid #808080;
			}
      .title{
      bottom: 5px;
      left: 5px;
      font-size: 16px;
      line-height: 19px;
      }
			img{
				top: 2px;
				right: 0;
			}
      .fa{
        top: 5px;
        right: 5px;
      }
		}
		nav{
			width: 272*100/320%;
		}
	}
}
.display{
	display: none;
}
</style>
