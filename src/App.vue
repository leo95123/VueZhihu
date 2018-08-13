<template>
    <div class="daily">
      <div class="daily-menu">
        <div class="daily-menu-item" :class="{on:type==='recommend'}"
              @click="handleToRecommend">每日推荐</div>
        <div class="daily-menu-item" :class="{on:type==='daily'}"
              @click="showThemes=!showThemes">主题日报</div>
        <ul v-show="showThemes">
            <li v-for="item in themes" :key="item">
              <a :class="{on:item.id==themeId&&type==='daily'}"
                  @click="handleToTheme(item.id)">{{item.name}}</a>
            </li>
        </ul>
      </div>
      <div class="daily-list">
        <template v-if="type==='recommend'">
          <div v-for="list in recommemdList" :key="list">
            <div class="daily-date">{{formatDay(list.date)}}</div>
            <Item v-for="item in list.stories" :key="item.id" :data='item'></Item>
          </div>
        </template>
        <template v-if="type==='daily'">
          <Item v-for="item in list" :key="item.id" :data="item"></Item>
        </template>
      </div>
      <daily-article></daily-article>
    </div>
</template>

<script>
import $ from './lib/util';
import Item from './components/item';
export default {
  name: 'App',
  components:{Item},
  data(){
    return{
      themes:[],
      showThemes:false,
      type:'recommend',
      list:[],
      themeId:0,
      recommemdList:[],
      dailyTime:$.getTodayTime(),
      isLoading:false
    }
  },
  methods:{
    getThemes(){
      $.ajax.get('themes').then(res=>{
        this.themes=res.others;
      })
    },
    handleToTheme(id){
      this.type='daily';
      this.themeId=id;
      this.list=[];
      $.ajax.get('theme/'+id).then(res=>{
        this.list=res.stories.filter(
          item=>item.type!==1
        )
      })
    },
    handleToRecommend(){
      this.type='recommend';
      this.recommemdList=[];
      this.dailyTime=$.getTodayTime();
      this.getRecommendList();
    },
    getRecommendList(){
      this.isLoading=true;
      const prevDay=$.prevDay(this.dailyTime+86400000);
      $.ajax.get('news/before/'+prevDay).then(res=>{
        this.recommemdList.push(res);
        this.isLoading=false;
      })
    },
    formatDay(date){
      let month=date.substr(4,2);
      let day=date.substr(6,2);
      if(month.substr(0,1)==='0'){
        month=month.substr(1,1);
      }
      if(day.substr(0,1)==='0'){
        day=day.substr(1,1);
      }
      return month+'月'+day+'日';
    }
  },
  mounted(){
    this.getThemes();
    this.getRecommendList();
  }
}
</script>

<style>
html,body{
  margin: 0;
  padding: 0;
  height: 100%;
  color: #657180;
  font-size: 16px;
}
.daily-menu{
  width: 150px;
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  overflow: auto;
  background: #f5f7f9;
}
.daily-menu ul {
  list-style: none;
}
.daily-menu ul li a{
  display: block;
  color: inherit;
  text-decoration: none;
  padding: 5px 0;
  margin: 5px 0;
  cursor: pointer;
}
.daily-menu ul li a:hover,.daily-menu ul li a .on{
  color:#3399ff;
}
.daily-menu-item{
  font-size: 18px;
  text-align: center;
  margin: 5px 0;
  padding: 10px 0;
  cursor: pointer;
  border-right: 2px solid transparent;
  transition: all .3s ease-in-out;
}
.daily-menu-item:hover{
  background: #e3e8ee;
}
.daily-menu-item .on{
  border-right: 2px solid #3399ff;
}
.daly-list{
  width: 300px;
  position: fixed;
  top: 0;
  bottom: 0;
  left: 150px;
  overflow: auto;
  border-right: 1px solid #d7dde4;
}
.daily-item{
  display: block;
  color:inherit;
  text-decoration: none;
  padding: 16px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease-in-out;
}
.daily-item:hover{
  background-color: #e3e8ee;
}
.daily-article{
  margin-left: 450px;
  padding: 20px;
}
</style>
