<template>
    <div style="min-height: 600px" v-loading="loading">
        <el-card shadow="never" style="min-height: 400px" v-if="blog.id">
            <div slot="header">
                <span>{{blog.title}}</span>
            </div>
            <div style="font-size: 0.8rem;line-height: 1.5;color: #f47983;border-bottom: 1px solid #E4E7ED;padding: 5px 0px 5px 0px">
                <pre style="font-family: '微软雅黑'">{{blog.description}}</pre>
            </div>
            <div v-html="blog.content" class="markdown-body" style="padding-top: 20px"></div>
             <div style="font-size: 0.7rem;line-height: 1.5; margin: 16px 0; color: #f47983;border-top: 2px solid #dfe2e5;padding: 5px 0px 5px 0px">
                发布 {{blog.createTime}} <br />
                更新 {{blog.updateTime}}
            </div>
        </el-card>
        <el-card shadow="never" style="margin-bottom: 20px;padding: 20px 0px 20px 0px;text-align: center" v-if="!blog.id">
            <font style="font-size: 30px;color:#dddddd ">
                <b>没有更新 ╮(๑•́ ₃•̀๑)╭</b>
            </font>
        </el-card>
    </div>
</template>
<script>
    import GistApi from '@/api/gist'
    export default {
        data() {
            return {
                query: {
                    page: 1,
                    pageSize: 1
                },
                loading: false,
                blog: {
                    id: "",
                    title: "",
                    content: "",
                    description: "",
                    createTime: "",
                    updateTime: ""
                }
            }
        },
        mounted() {
            this.loading = true
            GistApi.list(this.query).then((response) => {
                let result = response.data
                if (!result || result.length == 0) {
                    this.loading = false
                    return
                }
                for (let key in result[0].files) {
                    this.blog.id = result[0]['id']
                    break
                }
                GistApi.single(this.blog.id).then((response) => {
                    let result = response.data
                    for (let key in result.files) {
                        this.blog['title'] = key
                        this.blog['content'] = this.$markdown(result.files[key]['content'])
                        this.blog['description'] = result['description']
                        this.blog['createTime'] = this.$util.utcToLocal(result['created_at'])
                        this.blog['updateTime'] = this.$util.utcToLocal(result['updated_at'])
                        break
                    }
                }).then(() => this.loading = false)
            })
        },
    }
</script>