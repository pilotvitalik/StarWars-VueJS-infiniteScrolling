<template>
	<ul id='listCarts'>
		<li class='tst' v-for='page in peoples' :key='page.id'>
			<ul>
				<li v-for='people in page' :key='people.name' @click = 'descript(people)'>
					<span>{{people.name}}</span>
				      <ul>
						<li><img :src="people.image"/></li>
						<li>{{people.specie}}</li>
					  </ul>
				</li>
			</ul>
		</li>
	</ul>
</template>

<script>
	import {bus} from '../../../../../main.js'

	import luke_skywalker  from '../../../../../assets/common/peoples/luke_skywalker.png';
	import c_3po  from '../../../../../assets/common/peoples/c_3po.png';
	import r2_d2  from '../../../../../assets/common/peoples/r2_d2.png';
	import darth_vader  from '../../../../../assets/common/peoples/darth_vader.png';
	import leia_organa  from '../../../../../assets/common/peoples/leia_organa.png';
	import owen_lars  from '../../../../../assets/common/peoples/owen_lars.png';
	import beru_whitesun_lars  from '../../../../../assets/common/peoples/beru_whitesun_lars.png';
	import r5_d4  from '../../../../../assets/common/peoples/r5_d4.png';
	import biggs_darklighter  from '../../../../../assets/common/peoples/biggs_darklighter.png';
	import obi_wan_kenobi  from '../../../../../assets/common/peoples/obi_wan_kenobi.png';

export default{
	data(){
		return{
			img: [
				luke_skywalker,
				c_3po, r2_d2,
				darth_vader, 
				leia_organa,
				owen_lars,
				beru_whitesun_lars,
				r5_d4,
				biggs_darklighter,
				obi_wan_kenobi
			],
			peopleAPI: '',
			speciesAPI: [],
			page: 0,
			peoples: '',
		}
	},
	methods: {
		descript: function(people){
			bus.$emit('modalWindow', people)
		},
		fixData(tmpPeopleAPI){
			for(let i = 0; i < tmpPeopleAPI[0].length; i++){
								let string = '';
								string = tmpPeopleAPI[0][i].name.replace("-", "_").replace(" ", "_").replace(" ", "_").toLowerCase();
								tmpPeopleAPI[0][i].imgName = string;
								tmpPeopleAPI[0][i].image = '';
								tmpPeopleAPI[0][i].specie = '';
								let imageName = '';
								for(let u = 0; u < this.img.length; u++){
									imageName = this.img[u].slice(5, this.img[u].indexOf('.'));
									if(tmpPeopleAPI[0][i].imgName == imageName){
										tmpPeopleAPI[0][i].image = this.img[u]
									}
								}
							}
							this.peopleAPI = tmpPeopleAPI[0];
							let tmpSpecies = [];
							for(let j = 0; j < this.peopleAPI.length; j++){
								for(let k = 0; k < this.peopleAPI[j].species.length; k++){
									tmpSpecies.push(this.peopleAPI[j].species[k])
								}
							}
							let uniqSpecies = [];
							for(let item of tmpSpecies){
								if(!uniqSpecies.includes(item)){
									uniqSpecies.push(item)
								}
							}
							let counter = 0;
							tmpSpecies = [];
							let counterStatus = 0;
							for(let m = 0; m < uniqSpecies.length; m++){
								if(uniqSpecies[m] == undefined){
									counter;
								}else{
									counter++;
									this.$http.get(uniqSpecies[m]).then(response => {
										if(response.ok == true){
											counterStatus++
										}else{
											counterStatus
										};
										if(counter == counterStatus){
											bus.$emit('result', this.peopleAPI)
										}
										bus.$emit('counterStatus', counterStatus)
										tmpSpecies.push(response.body)
										bus.$emit('statusSpecies', response.ok)
										this.speciesAPI = tmpSpecies.map(item => {
											return {
												specie: item.name,
												url: item.url
											}
										})
										for(let n = 0; n < this.peopleAPI.length; n++){
											if(this.peopleAPI[n].species == ''){
												this.peopleAPI[n].specie = 'species uknown'
											}
											for(let k = 0; k < this.peopleAPI[n].species.length; k++){
												for(let q = 0; q < this.speciesAPI.length; q++){
													if(this.peopleAPI[n].species[k] == this.speciesAPI[q].url){
														this.peopleAPI[n].specie = this.speciesAPI[q].specie
													}
												}
											}
										}
									})
								}
							}
							bus.$emit('lengthSpecies', counter)
		},
		initialLoading(){
			let tmpPeopleAPI = [];
			this.$http.get('https://swapi.co/api/people/').then(response => {
				tmpPeopleAPI.push(response.body.results);
				bus.$emit('numberPage', response.body.count);
				this.fixData(tmpPeopleAPI);
			})
		},
		otherPages(page){
			let tmpPeopleAPI = [];
			this.$http.get('https://swapi.co/api/people/?page=' + page).then(response => {
				tmpPeopleAPI.push(response.body.results);
				this.fixData(tmpPeopleAPI);
			})
		},
		common(data){
			if(this.page != 0){
				this.otherPages(data)
			}
		}
	},
	created(){
		this.initialLoading()
		let sumStatus = 0;
		let statusArr = [];
		let uniqStatusArr = [];
		bus.$on('lengthSpecies', data => {
			bus.$on('statusSpecies', status => {
				sumStatus++;
				statusArr.push(status);
				if(data == sumStatus){
					for(let item of statusArr){
						if(!uniqStatusArr.includes(item)){
							uniqStatusArr.push(item)
						}
					}
					if(uniqStatusArr.length == 1){
						bus.$emit('closePreloader', uniqStatusArr[0])
						setTimeout(() => {
						                            document.querySelector('body').style.overflowY = 'auto';
						                            document.querySelector('body').style.pointerEvents = 'auto';
						                              let withScroll = document.documentElement.clientWidth
						                              let outScroll = window.innerWidth
						                              let body = document.querySelector('body')
						                              let headerTitle = document.querySelector('#Header>.logo')
						                              let left =  document.querySelectorAll('.left')
						                              let padding = document.querySelector('.padding');
						                              if(withScroll < outScroll){
						                                let delta = (outScroll - withScroll)*100/outScroll;
						                                let newDelta = parseFloat(delta.toFixed(2), 10);  
						                                body.style.width = (100+newDelta)+'%';
						                                body.style.overflowX = 'hidden';
						                                headerTitle.style.paddingLeft = -newDelta+'%';
						                                for(let i = 0; i < left.length; i++){
						                                  left[i].style.paddingLeft = -newDelta+'%';
						                                }
						                              }
						                              if(outScroll <= 767){
						                                body.style.width = outScroll
						                              }
						                              bus.$emit('showPages', true)
						                          }, 3000)
					}
				}
			})
		})
		bus.$on('currentPage', data => {
			this.page = data;
			this.common(data)
			console.log(this.page)
		});
		let com = [];
		bus.$on('result', (data) => {
			com.push(data)
			this.peoples = com;
			console.log(this.peoples)
		})
	},
	beforeMounte(){
		console.log('beforeMounte')
	},
	mounted(){
		console.log('Mounted')
	},
	beforeUpdate(){
		console.log('beforeUpdate')
	},
	updated(){
		console.log('updated')
	},
	beforeDestroy(){
		bus.$off('result');
		bus.$off('currentPage');
		console.log('beforeDestroy')
	},
}
</script>

<style lang='less' scoped>
#listCarts{
	display: block;
	position: relative;
	height: auto;
	.tst{
	display: block;
	position: relative;
	height: inherit;
		&>ul{
			display: flex;
			flex-direction: row;
			flex-wrap: wrap;
			justify-content: space-between;
			position: relative;
			height: auto;
			background: #333;
			&>li{
				display: flex;
				flex-direction: column;
				align-items: center;
				position: relative;
				border-radius: 8px;
				box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4);
				background: #1a1a1a;
				span{
					display: inline-block;
					position: absolute;
					color: #fff;
					font-family: 'Roboto', sans-serif;
					font-weight: bold;
				}
				ul{
					display: flex;
					flex-direction: column;
					align-items: center;
					justify-content: space-between;
					position: relative;
					li:first-child{
						display: block;
						position: relative;
						img{
							display: block;
							position: absolute;
							width: 100%;
							height: 100%;
							border-radius: inherit;
						}
					}
					li:last-child{
						display: inline-block;
						position: relative;
						color: #808080;
						font-family: 'Roboto', sans-serif;
					}
				}
			}
			&>li:hover{
				cursor: pointer;
			}
		}
	}
}
@media(min-width: 768px){
	#listCarts{
		width: 100%;
		.tst{
		width: 100%;
			&>ul{
				width: 100%;
				&>li{
					height: 320px;
					width: 592*100/1216%;
					margin-bottom: 32px;
					span{
						height: 21px;
						top: 183px;
						font-size: 18px;
						line-height: 21px;
					}
					ul{
						width: 95px;
						height: 135px;
						top: 93px;
						li:first-child{
							width: 80px;
							height: 80px;
							border-radius: 50%;
						}
						li:last-child{
							height: 15px;
							font-size: 13px;
							line-height: 15px;
						}
					}
				}
				&>li:hover{
					box-shadow: 0 10px 40px rgba(37, 136, 167, 0.38);
				}
			}
		}
	}	
}
@media(min-width: 320px) and (max-width: 767px){
	#listCarts{
		width: 100%;
		.tst{
		width: 100%;
			&>ul{
				flex-direction: column;
				flex-wrap: nowrap;
				align-items: center;
				justify-content: unset;
				width: 100%;
				&>li{
					height: 200px;
					width: 272*100/320%;
					margin-bottom: 24px;
					span{
						height: 19px;
						top: 123px;
						font-size: 16px;
						line-height: 19px;
					}
					ul{
						width: 90px;
						height: 135px;
						top: 33px;
						li:first-child{
							width: 80px;
							height: 80px;
							border-radius: 50%;
						}
						li:last-child{
							height: 14px;
							font-size: 12px;
							line-height: 14px;
						}
					}
				}
				&>li:hover{
					box-shadow: 0 10px 40px rgba(37, 136, 167, 0.38);
				}
			}
		}
	}	
}
</style>