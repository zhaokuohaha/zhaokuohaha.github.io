<template>
    <div>
        <section class="page-header" :style="getBgStyle()">
            <div style="position:absolute; top:20px; right:20px; z-index:2;">
                <el-tooltip effect="dark" :content="fullButton.full?'退出':'全屏'" placement="bottom-end">
                    <el-button @click="full" :icon="fullButton.full?'el-icon-close':'el-icon-rank'" circle></el-button>
                </el-tooltip>
            </div>
            <div v-for="(item,index) in randomIcon" :key="'ri'+index" :style="'position:absolute; top:'+item.top+'px; left:'+item.left+'px; z-index:1;'">
                <font :style="'font-size: '+item.size+'px;color:#fff;'">
                    <b>♪</b>
                </font>
            </div>
            <h1 class="project-name">{{blogTitle}}</h1>
            <h2 class="project-tagline">{{blogDescribe}}</h2>
            <a :href="'https://github.com/'+githubUsername" class="btn" target="_blank">GitHub主页</a>
        </section>
        <section class="main-content">
            <el-row>
                <div class="left-fixed">
                    <sidebar></sidebar>
                </div>
                <el-col :span="24" style="padding-left:10px">
                    <app-main></app-main>
                </el-col>
                <div class="right-fixed">
                    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=450 :src="'//music.163.com/outchain/player?type=0&id='+wyPlayListId+'&auto=0&height=430'"></iframe>
                </div>
            </el-row>
        </section>
        <section class="foot">
            <foot :user="githubUsername"></foot>
        </section>
    </div>
</template>
<script>
    import { mapGetters } from 'vuex'
    import Sidebar from './components/Sidebar'
    import AppMain from './components/AppMain'
    import Foot from './components/Foot'
    export default {
        components: {
            Sidebar,
            AppMain,
            Foot
        },
        data() {
            return {
                music: {
                    isPlay: false,
                    currentTime: 0,
                    maxTime: 0,
                    volume: 100
                },
                fullButton: {
                    full: false
                },
                topbar: {
                    active: "",
                },
                randomIcon: []
            }
        },
        computed: {
            ...mapGetters([
                'githubUsername',
                'blogTitle',
                'blogDescribe',
                'avatarUrl',
                'name',
                'location',
                'blog',
                'fontColor',
                'useBackgroundImage',
                'backgroundColorLeft',
                'backgroundColorRight',
                'audioUrl',
                'mini',
                'followersTotal',
                'followingTotal',
                'audioAutoPlay',
                'webSites',
                'wyPlayListId'
            ])
        },
        watch: {
            '$refs.music.currentTime': function () {
                console.log(this.$refs.music.currentTime)
            }
        },
        mounted() {
            this.$nextTick(() => {
                setInterval(this.listenMusic, 1000)
            })
            let width = window.innerWidth
            for (let i = 0; i < 12; i++) {
                let temp = {}
                let left = this.$util.randomInt(10, width - 310)
                if(left>width/2-150){
                    left+=300
                }
                temp["left"] = left
                temp["top"] = this.$util.randomInt(20, 300)
                temp["size"] = this.$util.randomInt(20, 40)
                this.randomIcon.push(temp)
            }
        },
        created() {

        },
        methods: {
            selectTopbar(index) {
                //取消菜单选中状态
                this.topbar.active = this.topbar.active == "" ? " " : ""
                switch (index) {
                    case "#githubHome":
                        window.open('https://github.com/' + this.githubUsername)
                        break
                    case "#blog":
                        if (this.blog) {
                            window.open((this.blog.match(/https?:\/\//i)?'':'https://') + this.blog)
                        } else {
                            this.$message({
                                message: '博主没有其他博客',
                                type: 'info'
                            })
                        }
                        break
                    default:
                        if(/#webSites-\d+/.test(index)){
                            let i = parseInt(index.split('-')[1])
                            let url = this.webSites[i].url
                            window.open((url.match(/https?:\/\//i)?'':'https://') + url)
                        }
                        break
                }
            },
            moveIcon(index) {
                let width = window.innerWidth
                this.randomIcon[index]["top"] = this.$util.randomInt(20, 300)
                let left = this.$util.randomInt(10, width - 310)
                if(left>width/2-150){
                    left+=300
                }
                this.randomIcon[index]["left"] = left
            },
            full() {
                if (!this.fullButton.full) {
                    this.$util.fullScreen()
                    this.fullButton.full = true
                } else {
                    this.$util.fullExit()
                    this.fullButton.full = false
                }
            },
            getBgStyle(){
                if(!this.useBackgroundImage)
                    return `background-image: linear-gradient(120deg,${this.backgroundColorLeft},${this.backgroundColorRight});color:${this.fontColor};`
            }
        }
    }
</script>

<style>
    .page-header {
        padding: 5rem 6rem;
        color: #fff;
        text-align: center;
        background-image: url('../../assets/banner.jpg');
        background-position: center;
        background-repeat: no-repeat;
        background-size: cover;
    }

    .project-name {
        font-size: 3.25rem;
        margin-top: 0;
        margin-bottom: 0.1rem;
    }

    .project-tagline {
        font-size: 1.25rem;
        margin-bottom: 2rem;
        font-weight: normal;
        opacity: 0.7;
    }

    .btn:hover {
        color: rgba(255, 255, 255, 0.8);
        text-decoration: none;
        background-color: rgba(255, 255, 255, 0.2);
        border-color: rgba(255, 255, 255, 0.3);
    }

    a:hover {
        text-decoration: underline;
    }

    a:active,
    a:hover {
        outline: 0;
    }

    .btn {
        padding: 0.75rem 1rem;
        display: inline-block;
        margin-bottom: 1rem;
        color: rgba(255, 255, 255, 0.7);
        background-color: rgba(255, 255, 255, 0.08);
        border-color: rgba(255, 255, 255, 0.2);
        border-style: solid;
        border-width: 1px;
        border-radius: 0.3rem;
        transition: color 0.2s, background-color 0.2s, border-color 0.2s;
    }

    a {
        color: #1e6bb8;
        text-decoration: none;
    }

    .btn+.btn {
        margin-left: 1rem;
    }

    .main-content {
        /* max-width: 80rem; */
        padding: 30px 0px 30px 0px;
        margin: 0 auto;
        font-size: 1.1rem;
        word-wrap: break-word;
        min-height: 800px;
        margin: 0 400px 0 300px;
        /* padding-left: 300px; */
    }

    .foot {
        max-width: 67rem;
        margin: 0 auto;
        font-size: 12px !important;
        color: #586069 !important;
        word-wrap: break-word;
    }

    .right-fixed{
        float: left;
        position: absolute;
        top: -10px;
        right: -330px;
    }

    .left-fixed {
        position: absolute;
        left: -210px;
        padding-right: 10px; 
        width: 200px;
    }
</style>