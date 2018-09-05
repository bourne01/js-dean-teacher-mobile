<template>
    <div>
        <div class="header">
            <div class="today-course-txt">今日您有<span>{{todayTotalClasses}}</span>堂课程</div>
            <!-- <span to="/course-list" class="all-courses" @click="onClick">{{allCourseText}}
                <img :src="require('../../assets/forward.png')" alt=""></span> -->
        </div>
        <div :style="classList"></div>
        <div class="class-info" v-if="isShow">
            <div class="class-time-sequence">
                <div class="class-sequence">
                    <span>1</span>
                    <div class="dashed-line"></div>
                </div>
                <div class="class-time">
                    <img :src="require('../../assets/time.png')" alt="">
                    &nbsp;&nbsp;&nbsp;{{todayFirstClass.classTime}}
                </div>                        
            </div>
            <div class="course">
                <div class="course-name">{{todayFirstClass.coursename}}</div>
                <div class="class">
                    <img :src="require('../../assets/classmate.png')" alt="">
                    &nbsp;<span class="item-text">班级</span>&nbsp;<span class="classname">{{todayFirstClass.classname}}</span>
                </div>
                <div class="classroom">
                    <img :src="require('../../assets/classroom.png')" alt="">
                    &nbsp;<span class="item-text">教室</span>&nbsp;<span class="classroom-name">{{todayFirstClass.classroom}}</span>
                </div>
                <div class="teacher">
                    <img :src="require('../../assets/teacher.png')" alt="">
                    &nbsp;<span class="item-text">老师</span>&nbsp;<span class="teacher-name">{{me.name}}</span>
                </div>
            </div>
            <!-- <exception @exception="getException" v-if="isException"></exception> -->
        </div>
        <div class="class-info" v-else v-for="(todayClass,index) in todayClasses" :key="index">
            <div class="class-time-sequence">
                <div class="class-sequence">
                    <span>{{index+1}}</span>
                    <div class="dashed-line"></div>
                </div>
                <div class="class-time">
                    <img :src="require('../../assets/time.png')" alt="">
                    &nbsp;&nbsp;&nbsp;{{todayClass.classTime}}
                </div>                        
            </div>
            <div class="course">
                <div class="course-name">{{todayClass.coursename}}</div>
                <div class="class">
                    <img :src="require('../../assets/classmate.png')" alt="">
                    &nbsp;<span class="item-text">班级</span>&nbsp;<span class="classname">{{todayClass.classname}}</span>
                </div>
                <div class="classroom">
                    <img :src="require('../../assets/classroom.png')" alt="">
                    &nbsp;<span class="item-text">教室</span>&nbsp;<span class="classroom-name">{{todayClass.classroom}}</span>
                </div>
                <div class="teacher">
                    <img :src="require('../../assets/teacher.png')" alt="">
                    &nbsp;<span class="item-text">老师</span>&nbsp;<span class="teacher-name">{{me.name}}</span>
                </div>
            </div>
            <exception @exception="getException" v-if="isException"></exception>
            
        </div>
        <no-data v-if="todayClasses.length===0&&!isException">今日无课</no-data>
    </div>
</template>

<script>
import Exception from './exception-popup'
import NoData from '../base/no-data'
export default {
    props:['schedule','teacher','term-id','is-exception'],
    components:{
        Exception,
        NoData,
    },
    data(){
        return{
            todayFirstClass:{classname:'',coursename:'',classroom:''},
            classTimeList:[
                            '08:20-09:00','09:10-09:50','10:05-10:45','10:55-11:35',
                            '13:20-14:00','14:10-14:50','15:05-15:45','15:45-16:25'],
            todayTotalClasses:0,//今日总共多少节课
            firstClassTime:'',//第一节上课时间
            todayClasses:[],//今日课程数
            allCourseText:'查看全部课程',
            classOrders:[],//记录有课的课序号
            classIds:[],//记录本课的授课班级Id
            isShow:false,
            me:{},//当前教师对象
            classList:{maxHeight:'',overflow:'auto'},
            //isException:false
            dir:'..'
        }
    },
    methods:{
        /**@function 监听查看课程事件 */
        onClick(){
            if(this.isShow == true)
                this.allCourseText = '收缩全部课程';
            else{
                this.allCourseText = '查看全部课程';
            }           
            this.isShow=!this.isShow;                        
        },
        /**@function 获取我的个人信息 */
        getMyInfo(){
            let url = this.dir+'/public/thr!myInf.action';
            let params = {};
            this.$http(url,{params})
                .then( res => {
                    let objData = res.data;
                    if(objData.success){
                        this.me = objData.data;
                        sessionStorage.setItem('Me',JSON.stringify(this.me));
                    }
                })
                .catch(err => {
                    this.reqErrorHandler(err);
                })
        },
        /**@function 从课表中获取今日第一节课 
        */
        getTodayClass(){
            //上课时间表
            let date = new Date();
            let today = date.getDay();//今日，值为0~6,0为星期天
            if(today ==0){
                today = 7; 
            }else{
                today = today - 1;
            }
            let isFirstClass = true;//是否今日头节课
            if(!this.schedule.length)return;
            for(let i=0;i<8;i++){//8为一天总共有多少节课
                //console.log(this.schedule[i][today]);
                if(this.schedule[i][today][0] && isFirstClass){
                    this.todayFirstClass.classname = this.schedule[i][today][0];
                    this.todayFirstClass.classroom = this.schedule[i][today][2];
                    this.todayFirstClass.coursename = this.schedule[i][today][1];
                    this.todayFirstClass.classTime = this.classTimeList[i];
                    this.todayFirstClass.courseId = this.schedule[i][today][4];
                    this.todayFirstClass.classId = this.schedule[i][today][3];
                    this.todayTotalClasses++;
                    isFirstClass = false;
                    this.todayClasses.push(this.todayFirstClass);
                    this.classOrders.push(i+1);
                }else if(this.schedule[i][today][0]){
                    this.todayTotalClasses++;
                    this.todayClasses.push({
                                            classname:this.schedule[i][today][0],
                                            classTime:this.classTimeList[i],
                                            classroom:this.schedule[i][today][2],
                                            coursename:this.schedule[i][today][1],
                                            courseId:this.schedule[i][today][4],
                                            classId: this.schedule[i][today][3],
                                        });
                    this.classOrders.push(i+1);
                }
            }
        },
        /**@function Ajax请求异常处理 
         * @param {出错对象} errObj
        */
        reqErrorHandler(errObj){
            //console.log(errObj);
            if(errObj.response){ 
                let errResStatus = errObj.response.status; 
                if(errResStatus == 500 || errResStatus == 504){
                    //this.$msgbox('网络异常','请稍后重试！',2000);
                    this.isException = true;
                }else if(errResStatus == 404){
                    //this.$router.push('/page-not/found');
                }else if(errResStatus == 401){
                    this.$msgbox('未授权登录,正在跳转...','',1000);
                    //this.$router.push('/login')
                    //location.href = 'http://my.wzzyzz.com/login?service='+location.href
            }}
        },       
        
        /**捕获网络异常事件 */
        getException(){
            //this.isException = false;
            this.$emit('exception');
            //console.log('Got an exception from network...')
        }
        
    },
    watch:{
            schedule:{
                handler:function(newVal, oldVal){
                    /* console.log(newVal);
                    console.log(oldVal);
                    console.log(this.schedule); */
                    this.getTodayClass();
                },
                deep:true,                
            }
        },
    mounted(){
        this.classList.maxHeight = 642/37.5+'rem';
        this.getTodayClass();
        this.getMyInfo();
    }
}
</script>

<style lang="scss" scoped>
    @function px2rem($px){
        $rem:37.5px;
        @return ($px / $rem) + rem; 
    }
    .today-course-txt{
        font-family: 'PingFang-SC-Heavy',"Microsoft YaHei";
        font-size:px2rem(28px);
        color:rgb(28,38,57);
    }
    .header{
        display:flex;
        justify-content: space-between;
        margin-bottom:px2rem(40px);
    }
    .all-courses{
        font-family: 'PingFang-SC-Medium',"Microsoft YaHei";
        font-size:px2rem(24px);
        color:rgb(155,158,163);
    }
    .header img{
        width:px2rem(40px);
        vertical-align: px2rem(-10px);
        
    }
    .class-info{
        position: relative;
    }
    .class-time-sequence{
        display:flex;
        justify-content: space-between;
    }
    .class-sequence{
        border-radius: px2rem(36px);
        background-color:#5c9dff;
        width:px2rem(36px);
        height:px2rem(36px);
        text-align: center;
        line-height:px2rem(36px);
        font-size:px2rem(24px);
        position: relative;
        color:#fff;
    }
    .dashed-line{
        width:0;
        height:px2rem(266px);
        border-left:1px dashed #9b9ea3;
        position: absolute;
        left:px2rem(18px);
    }
    .class-time{
        height:px2rem(36px);
        line-height:px2rem(36px);
        font-size:px2rem(24px);
        width:px2rem(594px);
        border-radius: px2rem(36px);
        background-color:rgb(251,245,243);
    }
    .class-time img{
        vertical-align: middle;
        width:px2rem(28px);
        margin-left:px2rem(40px);
    }
    .course{
        margin-left:px2rem(106px);
        height:px2rem(236px);
    }
    .course-name{
        margin-top:px2rem(30px);
        font-size:px2rem(28px);
        font-family: 'PingFang-SC-Heavy',"Microsoft YaHei";
        color:rgb(28,38,57);
    }
    .course img{
        width:px2rem(30px);
        vertical-align: middle;
    }
    .course .item-text{
        font-size:px2rem(24px);
        font-family: 'PingFang-SC-Medium',"Microsoft YaHei";
        color:rgb(163,170,185);
    }
    .classname,.classroom-name,.teacher-name{
        font-size:px2rem(24px);
        font-family: 'PingFang-SC-Medium',"Microsoft YaHei";
        color:rgb(28,38,57);
    }
</style>
