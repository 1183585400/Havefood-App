<template>
    <main>
        <mt-header class="t" title="写文章">
        <mt-button icon="back" slot="left" @click="back" class="l">返回</mt-button>
                <!-- <mt-button class="r" slot="right" @click="addnews">发布</mt-button> -->
        </mt-header>
        <section class="content">
            <div style="width:100%;padding-bottom：60%;">
                <nut-uploader
                    name="file"
                    url="/api/upload"
                    :isPreview="true"
                    :acceptType = "['image/jpeg', 'image/png', 'image/gif', 'image/bmp']"
                    @success="onSuccess"
                    @fail="onFail"
                    typeError="对不起，不支持上传该类型文件！"
                    limitError="对不起，文件大小超过限制！"
                    class="imgbox"
                >
                <!-- <video v-if="previewType == 'video'" :src="previewImg" width="100%" height="100%" controls="controls" /> -->
                <van-image fit="contain" v-if="previewImg"  width="100%" :src="previewImg" alt />
                <div v-else>
                    <van-icon name="plus" size="3em" color="#aaa"/>
                    <p style="color:#AAA;padding:1em 0">上传封面图片</p>
                </div>
                </nut-uploader> 
            </div>
            <div class="tit">
                <nut-textinput 
                    v-model="title"
                    placeholder="请输入文章标题"
                    :clearBtn="true"
                    :disabled="false"
                    :hasBorder="false"
                />
            </div>
            <div v-for="(item,index) in newsdata">
                <van-field v-if="item.sub !== undefined" v-model="item.sub" class="sub" placeholder="请输入二级标题">
                    <template #button>
                        <van-button size="small" icon="cross" type="default" @click="()=>{if(index==0) {item.sub = undefined;delete item.sub;} else newsdata.splice(index,1) }"></van-button>
                    </template>
                </van-field>
                <van-field
                    v-if="item.text !== undefined"
                    v-model="item.text"
                    rows="2"
                    type="textarea"
                    :autosize="{ maxHeight: 70, minHeight: 70 }"
                    placeholder="请输入段落内容"
                >
                    <template #button>
                        <van-button size="small" icon="cross" type="default" @click="()=>{if(index==0) {item.text = undefined;delete item.text;} else newsdata.splice(index,1) }"></van-button>
                    </template>
                </van-field>
                <div class="upimg" style="position:relative"  v-if="item.img !== undefined">
                    <van-uploader style="top: 10px;" :after-read="afterread" :max-count="9" :before-delete="beforedelete"  v-model="item.img"></van-uploader>
                    <van-button size="small" icon="cross" type="default" style="position:absolute;top:10px;right:16px" @click="()=>{if(index==0) {item.img = undefined;delete item.img;} else newsdata.splice(index,1) }"></van-button>
                </div>
            </div>
            <div class="btnlist">
                <span class="addbtn">
                    <van-icon name="orders-o" size="24px" style="display:block" @click="newsdata.push({sub:''})" />
                    +添加二级标题
                </span>
                <span class="addbtn">
                    <van-icon name="records" size="24px" style="display:block" @click="newsdata.push({text:''})" />
                    +添加段落文字
                </span>
                <span class="addbtn">
                    <van-icon name="photo-o" size="24px" style="display:block" @click="newsdata.push({img:[]})" />
                    +添加图片
                </span>
            </div>
            <nut-buttongroup style="position:fixed;bottom:0">
                <nut-button
                    type="light"
                    style="border-top:1px solid #ececee;width:100%"
                    @click="preview"
                    >
                        预览
                    </nut-button>
                    <nut-button @click="addNews" style="width:100%">
                        发布
                    </nut-button>
            </nut-buttongroup>
            
        </section>
        <van-popup v-model="show" position="bottom" closeable>
            <Prenews :show='show' :record = "record" ></Prenews>
        </van-popup>
    </main>
</template>

<script>
import { Toast, Dialog } from 'vant';
import Prenews from '../discovery/discory_pro/pre_news'
export default {
    components: { Prenews },
    data() {
        return {
            show:false,
            record:{},
            // previewType:'',
            // 上传图片
            previewImg: '',
            title:'',
            sub_title:'',
            message:'',
            fileList:[],
            newsdata:[{sub:'',text:'',img:[]}],
            flagnum1:'',
            flagnum2:'',
            initdata:''
        }
    },
    methods: {
        back(){
            // this.$router.go(-1)
            if(this.previewImg !== '' || this.title !== '' || this.newsdata[0].sub !== '' || this.newsdata[0].text !== '' || this.newsdata[0].img.length !== 0){
                Dialog.confirm({
                    title: '',
                    message: '是否保存到草稿？',
                    confirmButtonText:'是',
                    cancelButtonText:'否'
                }).then(() => {
                    for(var i=0;i<this.newsdata.length;i++){
                        if(this.newsdata[i].img){
                            for(var j=0;j<this.newsdata[i].img.length;j++){
                                this.newsdata[i].img[j] = this.newsdata[i].img[j].url
                            }
                        }
                    }
                    if(this.$store.state.newsdetails){
                        this.$axios.post('/sureNews',{
                            _id: this.$store.state.newsdetails._id,
                            cover_pic : this.previewImg, 
                            title : this.title, 
                            author : this.$store.state.userinfo.username, 
                            avatar : this.$store.state.userinfo.avatar,
                            media : this.newsdata, 
                            time : new Date(),
                            status : 2,
                        }).then(res => {
                            console.log(res.data)
                            setTimeout(()=>{
                                this.$router.go(-1)
                            },1200)
                        })
                    }else{
                        this.$axios.post('/addNews',{
                            cover_pic : this.previewImg, 
                            title : this.title, 
                            author : this.$store.state.userinfo.username, 
                            avatar : this.$store.state.userinfo.avatar,
                            media : this.newsdata, 
                            time : new Date(),
                            status : 2,
                        }).then(res => {
                            console.log(res.data)
                            setTimeout(()=>{
                                this.$router.replace({path:'/recommend'})
                            },1200)
                        })
                    }
                    
                }).catch(() => {
                    if(this.$store.state.newsdetails){
                        this.$axios.post('/deleteNews',{
                            _id: this.$store.state.newsdetails._id,
                        }).then(res=>{
                            console.log('删除草稿成功')
                        })
                    }
                    this.$router.go(-1)
                });
                    
            }else{
                this.$router.go(-1)
            }
        },
        // 上传素材
        onSuccess(file,res){
            let resop = JSON.parse(res)
            this.previewImg = resop.result.url
            Toast({
                message:'上传成功',
                position:"bottom"
            });
            // this.previewType = resop.type
        },
        onFail(file,res){
            Toast({
                message:'上传失败！',
                position:"bottom"
            });
        },
        // 上传步骤图片
        afterread(file){
            // console.log(index,'索引')
            console.log(file,'这是什么')
            let params = new FormData(); //创建form对象
            params.append("file", file.file); //通过append向form对象添加数据//第一个参数字符串可以填任意命名，第二个根据对象属性来找到file
            let config = {
                headers: { //添加请求头
                Authorization:
                    "Bearer " + window.localStorage.getItem("managementToken"),
                    "Content-Type": "multipart/form-data"
                }
            };
            let url = "";
            this.$axios.post('/upload',params,config).then(res=>{
                for(let i=0;i<this.newsdata.length;i++){ 
                    console.log(this.newsdata[i],'img0')
                    if(this.newsdata[i].img){
                        for(let j=0;j<this.newsdata[i].img.length;j++){
                            if(file.file.name == this.newsdata[i].img[j].file.name){
                                this.newsdata[i].img[j].url = res.data.result.url
                            }
                        }
                    }
                    
                }
                // console.log(this.newsdata,'[i].img[j]')
            })
        },
        // 删除步骤
        beforedelete(file){
            console.log(file,'删除的file')
            for(let i=0;i<this.newsdata.length;i++){ 
                if(this.newsdata[i].img){
                    for(let j=0;j<this.newsdata[i].img.length;j++){
                        if(file.file.name == this.newsdata[i].img[j].file.name){
                            this.flagnum1 = i
                            this.flagnum2 = j
                        }
                    }
                }
            }
            this.$axios.post('/deleteImg',{
                url:this.newsdata[this.flagnum1].img[this.flagnum2].url
            }).then(res => {
                this.newsdata[this.flagnum1].img.splice(this.flagnum2,1)
            })
        },
        preview(){
            if(this.previewImg == ''){
                Toast({
                    message:'请先上传封面图',
                    position:"bottom"
                });
            }else if(this.title == ''){
                Toast({
                    message:'请输入文章标题',
                    position:"bottom"
                });
            }else if(this.newsdata[0].sub == '' && this.newsdata[0].text == '' && this.newsdata[0].img == ''){
                Toast({
                    message:'请输入文章内容',
                    position:"bottom"
                });
            }else{
                this.show = true
                var myDate = new Date();
                var time = myDate.toLocaleString();
                this.record['cover_pic'] = this.previewImg
                this.record['title'] = this.title
                this.record['author'] = this.$store.state.userinfo.username, 
                this.record['avatar'] = this.$store.state.userinfo.avatar, 
                this.record['media'] = this.newsdata,
                this.record['time'] = time
            }
        },
        addNews(){
            // console.log(this.newsdata,'这是新闻')
            var time = new Date();
            if(this.previewImg == ''){
                Toast({
                    message:'请先上传封面图',
                    position:"bottom"
                });
            }else if(this.title == ''){
                Toast({
                    message:'请输入文章标题',
                    position:"bottom"
                });
            }else if(this.newsdata[0].sub == '' && this.newsdata[0].text == '' && this.newsdata[0].img == ''){
                Toast({
                    message:'请输入文章内容',
                    position:"bottom"
                });
            }else{
                for(var i=0;i<this.newsdata.length;i++){
                    if(this.newsdata[i].img){
                        for(var j=0;j<this.newsdata[i].img.length;j++){
                            this.newsdata[i].img[j] = this.newsdata[i].img[j].url
                        }
                    }
                }
                this.$axios.post('/addNews',{
                    cover_pic : this.previewImg, 
                    title : this.title, 
                    author : this.$store.state.userinfo.username, 
                    avatar : this.$store.state.userinfo.avatar,
                    media : this.newsdata, 
                    time : time,
                    status : 1,
                }).then(res => {
                    console.log(res.data)
                    setTimeout(()=>{
                        this.$router.replace({path:'/recommend'})
                    },1200)
                })
            }
        },
        getInit(){
            this.initdata = this.$store.state.newsdetails
            if(this.initdata.cover_pic){
                this.previewImg = this.initdata.cover_pic
            }
            if(this.initdata.title){
                this.title = this.initdata.title
            }
            if(this.initdata.media && this.initdata.media.length !== 0){
                let arr = []
                for(var i=0;i<this.initdata.media.length;i++){
                    arr = []
                    if(this.initdata.media[i].img){
                        for(var j=0;j<this.initdata.media[i].img.length;j++){
                            arr.push({url : this.initdata.media[i].img[j]})
                        }
                    this.initdata.media[i].img = arr
                    }
                }
                this.newsdata = this.initdata.media
            }
            console.log(this.initdata,'initdata')
        }
    },
    mounted() {
        this.getInit()
    },
    destroyed() {
        this.$store.commit('getnewsdetails',{newsdetails:''}) 
        window.localStorage.removeItem('newsdetails')
        // this.$store.commit('recipedetails','')  //置空detail
    },
}
</script>

<style scoped>
 main{
        width: 100%;
        height: auto;
        /* background-color: #f5f5f5; */
    }
	.t{
		height: 50px;
	    background: #fdfdfd;
	    padding: 0px 16px;
	    color: #000;
	    font-size: 18px;
        width: 100%;
        position: fixed;
        z-index: 1;
	}
	/* .l{
	    background: url(../assets/images/back_icon.png) left center no-repeat;
    	background-size: 19px 44px;
    	padding-left: 16px;
	} */
	.l,.r{
		color: #ff5151;
		font-size: 16px;
    }
    /* 封面素材 */
    .imgbox{
        padding-bottom:60%;
        overflow: hidden;
    }
    .imgbox img,.imgbox video{
        position:absolute;
        top:0;
        bottom:0;
        left:0;
        right:0;
        width:100%;
    }
    .imgbox div{
        position:absolute;
        top:50%;
        left: 50%;
        transform: translate(-50%,-50%);
    }
    /*网页内容*/
	.content{
		padding-top: 50px;
		padding-bottom: 50px;
	}
    .tit >>> .nut-textinput input{
		height: 3.5em !important;
		font-size: 16px;
        background-color: #f5f5f5;
        /* color: #aaaaaa; */
    }
    .tit >>> .nut-textinput input::-webkit-input-placeholder {
      /* color: #ffffff !important; */
      font-size: 16px !important; 
    }
    .upimg >>> .van-uploader{
        display: block;
    }
    .upimg >>> .van-uploader__preview ,.content >>> .van-uploader__upload{
        margin: 0 0 8px 16px;
    }
    .btnlist{
        margin: 20px 0 50px 0;
    }
    .addbtn{
        font-size:12px;
        text-align:center;
        display:inline-block;
        width:30%;
        color:#ff5151;
        line-height: 24px;
    }
    
</style>