<template>
    <div class="w-25">
        <input v-model="title" type="text" class="mb-3 form-control" placeholder="Title">

        <div ref="dropzone" class="mb-3 p-5 bg-dark text-light text-danger">
            Upload
        </div>

        <input @click.prevent="upload" class="btn btn-success" type="submit" value="upload">

        <div class="mb-3">
            <vue-editor v-model="content" useCustomImageHandler @image-added="imageHandler"></vue-editor>
        </div>

        <div class="mt-5">
            <div v-if="post">
                <h4>{{post.title}}</h4>
                <div v-for="image in post.images" class="mb-3">
                    <img :src="image.url" class="mb-3">
                    <img :src="image.preview_url" >
                </div>
                <div class="ql-editor" v-html="post.content"></div>
            </div>
        </div>
    </div>
</template>

<script>
    import Dropzone from 'dropzone'
    import {VueEditor} from "vue2-editor"

    export default {
        name: "Index",
        data: () => ({
            dropzone: null,
            title: null,
            post: null,
            content: "<h1>Some text ...</h1>"
        }),
        components: {
            VueEditor
        },
        methods: {
            imageHandler(file, Editor, cursorLocation, resetUploader){
                const formData = new FormData();
                formData.append("image", file);

                axios.post('/api/posts/images', formData).then(res => {
                    const url = res.data.url;
                    Editor.insertEmbed(cursorLocation, "image", url);
                    resetUploader();
                    console.log(res)
                })
            },
            upload(){
                let data = new FormData();
                this.dropzone.getAcceptedFiles().forEach(item => {
                    data.append('images[]', item);
                    this.dropzone.removeFile(item);
                })

                data.append('title', this.title);
                data.append('content', this.content);

                this.title = ''
                this.content = ''
                axios.post('/api/posts', data).then(res => {
                    this.getPost()
                })
            },
            getPost(){
                axios.get('/api/posts').then(res => {
                    this.post = res.data.data
                })
            }
        },
        mounted() {
            this.dropzone = new Dropzone(this.$refs.dropzone, {
                url: "/api/posts",
                autoProcessQueue: false,
                addRemoveLinks: true
            })

            this.getPost();
        }
    }
</script>

<style scoped>

</style>
