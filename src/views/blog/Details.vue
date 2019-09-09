<template>
    <div style="min-height: 600px" v-loading="loading">
        <el-card shadow="never" style="min-height: 400px">
            <div slot="header">
                <el-row>
                    <el-col :span="12">
                        <span>{{blog.title}}</span>
                    </el-col>
                    <el-col :span="12">
                        <div style="text-align: right;">
                            <el-button @click="$share()" style="padding: 3px 0" type="text" icon="el-icon-share">分享</el-button>
                            <el-button @click="edit" style="padding: 3px 0" type="text" icon="el-icon-edit" v-if="token">编辑</el-button>
                            <el-button style=" padding: 3px 0" type="text" icon="el-icon-more-outline" @click="more">更多博客</el-button>
                        </div>
                    </el-col>
                </el-row>
            </div>
            <div style="font-size: 0.8rem;line-height: 1.5; color: #75878a">
                原文链接: <a :href="blogurl">{{blogurl}}</a> <br>
                GIST链接: <a :href="gisturl">{{gisturl}}</a>
            </div>
            <div style="font-size: 0.8rem;line-height: 1.5;color: #f47983;border-bottom: 1px solid #E4E7ED;padding: 5px 0px 5px 0px">
                <pre style="font-family: '微软雅黑'">{{blog.description}}</pre>
            </div>
            <div v-html="blog.content" class="markdown-body" style="padding-top: 20px">
            </div>
            <div style="font-size: 0.7rem;line-height: 1.5; margin: 16px 0; color: #f47983;border-top: 2px solid #dfe2e5;padding: 5px 0px 5px 0px">
                发布 {{blog.createTime}} <br />
                更新 {{blog.updateTime}}
            </div>
            <div class="comment">
                <div id="container"></div>
            </div>
        </el-card>
    </div>
</template>
<script>
    function renderComment(conf, title) {
        var gitment = new Gitment({
            id: location.href.split('/').slice(-1)[0],
            owner: conf.owner,
            repo: conf.repo,
            oauth: {
                client_id: conf.clientId,
                client_secret: conf.clientSecret,
            },
            title: title
        })
        gitment.render('container')
    }
    import { mapGetters } from 'vuex'
    import GistApi from '@/api/gist'
    export default {
        data() {
            return {
                blog: {
                    id: "",
                    title: "",
                    content: "",
                    description: ""
                },
                loading: false,
            }
        },
        computed: {
            ...mapGetters([
                'token',
                'gitment',
                'githubUsername'
            ])
        },
        mounted() {
            this.loading = true
            this.blog.id = this.$route.params.id
            this.blogurl = window.location
            this.gisturl = `https://gist.github.com/${this.githubUsername}/${this.blog['id']}`

            console.log(this.$route.params)
            GistApi.single(this.blog.id).then((response) => {
                let result = response.data
                for (let key in result.files) {
                    this.blog['title'] = key
                    this.blog['content'] = this.$markdown(result.files[key]['content'])
                    this.blog['description'] = result['description']
                    this.blog['createTime'] = this.$util.utcToLocal(result['created_at'])
                    this.blog['updateTime'] = this.$util.utcToLocal(result['updated_at'])
                    // console.log(JSON.stringify(this.blog))
                    break
                }
            }).then(() => {
                this.loading = false
                let title = this.blog['title']
                document.title = title
                if(this.gitment){
                    renderComment(this.gitment, title)
                }
            })
        },
        methods: {
            edit() {
                if (!this.token) {
                    this.$message({
                        message: '请绑定有效的Token',
                        type: 'warning'
                    })
                    return
                }
                this.$router.push('/user/blog/edit/' + this.blog.id)
            },
            more() {
                this.$router.push('/user/blog/main')
            }
        }
    }
</script>