<template>
    <div style="position:relative">        
        <div class="top">
            <div class="slogan">
                越努力，越幸运
            </div>
            <img :src="require('../assets/books-startup.png')">            
        </div>
        <div class="middle"></div>
        <div class="bottom">
            <img :src="require('../assets/logo.png')" alt="">
        </div>
        <div class="count-down" @click="skip">{{countDown}}s&nbsp;跳过</div>
    </div>
</template>

<script>
export default {
    data(){
        return{
            countDown:3,//倒计时初始值3s
        }
    },
    methods:{
        /**@function 监听用户点击跳过事件 */
        skip(){
            this.$router.push('/login');
            clearInterval(this.timer)
        },
        /**@function 根据不同的分辨计算div.top和div.bottom盒子高度*/
        setDivTopHeight(){
            let divTop = document.querySelector(".top");
            let divBottom = document.querySelector(".bottom");
            let htmlHeight = document.documentElement.clientHeight || document.body.clientHeight;
            let htmlWidth = document.documentElement.clientWidth || document.body.clientWidth;
            let divTopHeight = (htmlHeight*460/667) + 'px';       
            divTop.style.height = divTopHeight;
            divBottom.style.height = htmlHeight - (htmlHeight*460/667) + "px";//高度与行高相等以使图片垂直居中
            divBottom.style.lineHeight = divBottom.style.height;
        }
    },
    mounted(){
        this.setDivTopHeight();
        /**@function 倒计时3秒跳转到登录页面*/
        this.timer = setInterval(()=>{
            this.countDown--;
            if(this.countDown === 0){
                clearInterval(this.timer);
                this.$router.push('/login');
            }                
        },1000)
    }
}
</script>

<style lang="scss" scoped>
    @function px2rem($px){
        $rem:37.5px;
        @return ($px / $rem) + rem; 
    }
    .top img{
        width:px2rem(206px);position:absolute;
        bottom:0;
        left:px2rem(272px);
    }
    .slogan{
        font-size:px2rem(48px);
        font-weight: bold;
        color:#fff;
        text-align: center;
        padding-top:px2rem(298px);
    }
    .top{
        background-image:url("../assets/startup-bg.png");
        background-size:contain;
        position: relative;
    }
    .middle{
        height:px2rem(48px);
        background-color:#3e5b97;
        }
    .bottom{
        text-align: center;
    }
    .bottom img{
        width:px2rem(350px);        
    }
    .count-down{
        position: absolute;
        right:px2rem(40px);
        top:px2rem(40px);
        height:px2rem(48px);
        line-height:px2rem(48px);
        width:px2rem(140px);
        border-radius: px2rem(24px);
        text-align: center;
        background-color: rgba(0,0,0, 0.2);
        color:#fff;
        font-size:px2rem(24px);
        font-family: 'pingfang sc regular';
    }
</style>

