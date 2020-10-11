<template>
<div>
	<div class="home">
		<!--1.导航栏-->
		<mt-header title="推推小说" class="nav-bar"></mt-header>
		
		<!--2.轮播图-->
      <div class="banner">
        <swipe :auto="4000">
          <swipe-item v-for="(item,index) in banner" :key="index">
            <img :src="item" alt="banner" class="banner-item">
          </swipe-item>
        </swipe>
      </div>
     	
     	<!--3.搜索框-->
      <div class="search">
        <div class="search-con">
          <input type="text" :placeholder="placeholder" class="input" v-model="search">
        </div>
        <div class="search-btn" @click="goSearch()">
          <i class="fa fa-search"></i>
        </div>
      </div>
      
      	<!--4.展示-->
    <div class="bookLists">  	
    	<div class="select">
      		<ul class="select-bar">
        		<li v-for="(item,index) in largeTypes" :class="{'active':index===largeTypeIndex}"
            		@click="setLargeType(item.type,index,item.duration)">{{item.name}}
        		</li>
      		</ul>
      		<ul class="select-bar">
        		<li v-for="(item,index) in smallTypes" :class="{'active':index===smallTypeIndex}"
            		@click="setSmallType(item.minor,index)">{{item.name}}
        		</li>
      		</ul>
    	</div>
    	<div class="load-more">
      		<mt-loadmore class="loadmore" :top-method="loadTop" :bottom-method="loadBottom" ref="loadmore" :auto-fill="false"
                   :bottom-all-loaded="allLoaded">
        		<div class="book-list">
          			<book-list :book="book" v-for="book in bookLists" :key="book._id" :isBookList="isBookList"
                     :isLoadMore="isLoadMore"></book-list>
        		</div>
      		</mt-loadmore>
    	</div>
    </div>  
      
      
	</div>
		<router-view></router-view>
</div>
</template>

<script type="text/ecmascript-6">
	import "mint-ui/lib/style.css" 
	import BookList from 'common/book-list'
  	import {Indicator} from 'mint-ui'
  	import {bookLists} from  '../../api/api'
	import {mapMutations} from 'vuex'
	export default {
		name:'home',
		components:{
			BookList
		},
		data(){
			return {
				 banner: [
				 	'http://statics.zhuishushenqi.com/recommendPage/153139406636473',
          			'http://statics.zhuishushenqi.com/recommendPage/15313940968984',
          			'http://statics.zhuishushenqi.com/recommendPage/153139416791777',
          			'http://statics.zhuishushenqi.com/recommendPage/153139401503529'
          			],
          		largeTypeIndex: 0,
        		smallTypeIndex: 0,
        		type: 'collectorCount',
        		duration: 'last-seven-days',
        		minor: '',
        		gender: '',
        		search: '',
        		start: 0,
        		currentPage: 1,
                largeTypes: [
          			{
            			type: 'collectorCount',
            			name: '本周最热',
            			duration: 'last-seven-days'
          			},
          			{
            			type: 'created',
           				name: '最新发布',
            			duration: 'all'
          			},
          			{
            			type: 'collectorCount',
            			name: '最多收藏',
            			duration: 'all'
          			}
        		],
        smallTypes: [
          {
            minor: '',
            name: '全部'
          },
          {
            minor: 'male',
            name: '男生',
          },
          {
            minor: 'female',
            name: '女生'
          }
        ],	
                    bookLists: [],
        			allLoaded: false,
        			isBookList: true,
        			isLoadMore: false,
                	placeholder: '圣墟',
        			home: true,
        
			}
		},
		created(){
      		this._getBookLists(this.type, this.duration, this.start, this.gender)			
		},
		methods:{
      		setLargeType(type, index, duration){
        		this.largeTypeIndex = index;
        		this.type = type;
        		this.duration = duration;
        		this._getBookLists(this.type, this.duration, this.start, this.gender)
      		},
      		setSmallType(minor, index){
        		this.smallTypeIndex = index;
        		this.minor = minor;
        		this.gender = '&gender=' + minor;
        		this.currentPage = 1;
        		this._getBookLists(this.type, this.duration, this.start, this.gender)
      		},	
      		_getBookLists(type, dur, start, gender){
        		Indicator.open('加载中');
        		bookLists(type, dur, start, gender).then((res)=>{
            		if(res.data.ok){
              		this.bookLists = this._unEscape(this.normalizeBooks(res.data.bookLists))
              		Indicator.close();
            		}
        		},(error)=>{
          		Indicator.close();
          		Indicator.open('网络错误');
           		setTimeout(()=>{
             		Indicator.close();
           		},1500)
        		})
//        api.get(`/booklists?sort=${type}&duration=${dur}&start=${start}${gender}`, {}, (res) => {
//          this.bookLists = this._unEscape(this.normalizeBooks(res.bookLists))
//          Indicator.close();
//        })
      		},      		
      		_unEscape(arr){
        		for (let i = 0; i < arr.length; i++) {
          		arr[i].cover = unescape(arr[i].cover);
          		arr[i].cover = arr[i].cover.replace("/agent/", "")
        		}
        		return arr
      		},      		
      		loadBottom(){
        		//加载更多
        		this.isLoadMore = true;
        		Indicator.open('加载中');
        		api.get(`/booklists?sort=${this.type}&duration=${this.duration}&start=${this.currentPage * 20}${this.gender}`, {}, (res) => {
          		res.bookLists.cover = this._unEscape(this.normalizeBooks(res.bookLists));
          		this.bookLists = [...this.bookLists, ...res.bookLists];
          		this.currentPage++;
          		setTimeout(() => {
            		this.isLoadMore = false
            		Indicator.close();
          		}, 350)
        		});
        		this.$refs.loadmore.onBottomLoaded()
      		},
      		loadTop(){
        		//下拉加载
        		this._getBookLists(this.type, this.duration, this.start, this.gender);
        		this.$refs.loadmore.onTopLoaded()
      		},     	
      		goSearch(){
        		if (!this.search) {
          		this.$router.push({
            		path: '/search',
            		query: {
              		val: this.placeholder
            		}
          		});
          		this.setSearch(this.placeholder)
        		} else {
          		this.$router.push({
            		path: '/search',
           	 		query: {
              		val: this.search
            		}
          		});
          		this.setSearch(this.search)
        		}

      		},
      		...mapMutations({
        		setSearch: 'SET_SEARCH'
      		}),      	
      	
      	
      	
		}
		
	
	}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  .banner
    width 100%
    height 3.2rem
    .banner-item
      width 100%
      height 100%
  .search
    padding 15px 18px
    position relative
    .search-con
      border 1px solid #d6d6d6; /* no*/
      overflow hidden
      .input
        line-height 50px
        padding 0 10px
        width 100%
        box-sizing border-box
        outline none
    .search-btn
      position absolute
      right 18px
      top 15px
      width 1.5rem
      height 50px
      background-color #ff8198
      color #fff
      border 1px solid #ff8198;
      line-height 50px
      text-align center

 
     
.bookLists   
    .select {
      position absolute
     
      left 0
      background #fff
      z-index 10
      width 100%
    }
    .select-bar {
      display flex
      flex-direction row
      justify-content flex-start
      flex-wrap nowrap
      height 100px
      width 100%
      overflow-x auto
      overflow-y hidden
      border-bottom 1px solid #f2f2f2
    }
    .select-bar li {
      flex-shrink 0
      line-height 100px
      padding 0 20px
      font-size 25px
    }
    .active {
      color red
    }
    
    .book-list {
      width 100%
      background #fff
    }

.home
	.nav-bar 
      height 100px
      line-height 100px
      font-size 25px
      background #ff8198      
</style>