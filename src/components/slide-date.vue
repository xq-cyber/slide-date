<template>
  <div :style="{display:'grid','grid-template-columns':`repeat(${dataList.length},1fr)`,padding:'0 32px','user-select':'none'}">
    <div style="position:relative;" v-for="(listItem,index) in dataList" :key="index">
      <div :id="`scroll-wheel-${index === 0 ? 'first' : (index === 1 ? 'second' : 'third')}`" class="scroll-wheel" :style="{height:height+'px'}" @scroll="scrollFn($event,listItem)">
        <ul :style="{'padding-top':((height/2)-(itemHeight/2))+'px','padding-bottom':((height/2)-(itemHeight/2))+'px'}">
          <li class="scroll-item" v-for="(item,index) in listItem" :key="index" :style="{height:itemHeight+'px',lineHeight:itemHeight+'px'}">{{item}}</li>
        </ul>
      </div>
      <div class="mask">
        <div class="top-line" :style="{height:'calc(50% - '+(itemHeight/2)+'px)'}"></div>
        <div class="bottom-line" :style="{height:'calc(50% - '+(itemHeight/2)+'px)'}"></div>
      </div>
    </div>
  </div>
</template>

<script>
  export default{
    props:{
      value:String,
      height:{
        type:[Number,String],
        default:240,
      },
      itemHeight:{
        type:[Number,String],
        default:40,
      },
    },  
    data(){
      return {
        timer:null,
        selectYear:1970,
        selectMonth:1,
        selectDate:1,
        t1:0,
        t2:0,
        dataList:[],
        initYearList:[],
        initMonthList:[1,2,3,4,5,6,7,8,9,10,11,12],
        initDateList:[]
      }
    },
    mounted(){
      const thisYear = new Date().getFullYear()+20
      for(let i=1970;i<=thisYear;i++){
        this.initYearList.push(i)
      }
      for(let i=1;i<=31;i++){
        this.initDateList.push(i)
      }
      this.dataList = [this.initYearList,this.initMonthList,this.initDateList]
      // 设置传入的年月日
      const modelValue = this.value || new Date()
      const year = new Date(modelValue).getFullYear(),
            month = new Date(modelValue).getMonth()+1,
            date = new Date(modelValue).getDate()
      this.$nextTick(()=>{
        this.selectYear = this.initPosition(this.initYearList,year,'scroll-wheel-first',this.itemHeight)
        this.selectMonth = this.initPosition(this.initMonthList,month,'scroll-wheel-second',this.itemHeight)
        this.selectDate = this.initPosition(this.initDateList,date,'scroll-wheel-third',this.itemHeight)
        this.$emit("getDate",{
          year:this.selectYear,
          month:this.selectMonth,
          date:this.selectDate,
          DATE:`${this.selectYear<10?'0'+this.selectYear:this.selectYear}-${this.selectMonth<10?'0'+this.selectMonth:this.selectMonth}-${this.selectDate<10?'0'+this.selectDate:this.selectDate}`
        })
      })
    },
    computed:{
      
    },   
    methods:{
      initPosition(list,compare,eId,itemHeight){
        let result;
        list.map((item,index)=>{
          if(item === compare){
            // scrollTo失效 暂未发现原因 可能由于样式未设置就触发的原因
            document.getElementById(eId).scrollTop = index*itemHeight
            result =  compare
          }
        })
        return result;
      },
      scrollFn(e,list){
        const {clientHeight,scrollHeight,scrollTop} = e.target
        const position = scrollTop/this.itemHeight
        const mathRound = Math.round(position)
        // 判断是否正在滚动
        let t1 = this.t1,
            t2 = this.t2,
            itemHeight = this.itemHeight

        clearTimeout(this.timer)
        t1 = scrollTop
        this.timer = setTimeout(()=>{isScrollEnd()},10)
        const isScrollEnd = () => {
          t2 = scrollTop
          if(t2 === t1){
            list.map((item,index)=>{
              if(mathRound === index){
                const id = e.target.id
                document.getElementById(id).scrollTo({top: mathRound*itemHeight, behavior: 'smooth'})
                switch(id){
                  case "scroll-wheel-first":
                    this.selectYear = list[index]
                    break;
                  case "scroll-wheel-second":
                    const year = new Date().getFullYear()
                    const dateList = [],monthDay = new Date(year,item,0).getDate()
                    for(let i=1;i<=monthDay;i++){
                      dateList.push(i)
                    }
                    this.initDateList = dateList
                    this.dataList = [this.initYearList,this.initMonthList,dateList]
                    this.selectMonth = list[index]
                    break;
                  case "scroll-wheel-third":
                    this.selectDate = list[index]
                    break;
                }
              }
            })
            this.$emit("getDate",{
              year:this.selectYear,
              month:this.selectMonth,
              date:this.selectDate,
              DATE:`${this.selectYear<10?'0'+this.selectYear:this.selectYear}-${this.selectMonth<10?'0'+this.selectMonth:this.selectMonth}-${this.selectDate<10?'0'+this.selectDate:this.selectDate}`
            })
          }
        }
      }
    }
  }
</script>

<style scoped>
*{
  margin:0;
  padding:0;
}
li{list-style: none;}
.scroll-wheel{
  position:relative;
  overflow-y:scroll;
  user-select: none;
}
.scroll-wheel::-webkit-scrollbar{
  display:none;
}
.mask{
  position:absolute;
  top:0;
  left:0;
  width:100%;
  height:100%;
  pointer-events: none;
}
.top-line{
  position:absolute;
  left:0;
  top:-1px;
  width:100%;
  background: linear-gradient(to bottom, rgba(255,255,255,1) 0%, rgba(255,255,255,.5));
  pointer-events: none;
  border-bottom:1px solid #eee;
}
.bottom-line{
  position:absolute;
  left:0;
  bottom:-1px;
  width:100%;
  background: linear-gradient(to top, rgba(255,255,255,1) 0%, rgba(255,255,255,.5));
  pointer-events: none;
  border-top:1px solid #eee;
}
</style>