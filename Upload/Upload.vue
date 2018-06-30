<template>
    <div class="upload-container">
        <!-- 预览 -->
        <div class="preview-wrap" v-show="showImg">
            <img :src="dataUrl" alt="" class="preview" :value="imgValue">
            <span class="el-upload-list__item-actions">
                <span class="el-upload-list__item-delete" @click="handleDel">
                    <i class="el-icon-delete"></i>
                </span>
            </span>  
        </div>
        <!-- 裁剪 -->
        <div class="cropper-wrap" v-show="showCropper">
            <vueCropper
                ref="cropper"
                :img="dataUrl"
                :outputSize="1"
                :autoCrop="true"
                :autoCropWidth="128"
                :autoCropHeight="128"
                :fixedBox="true"
            ></vueCropper>
        </div>
        <!-- 上传 -->
        <div class="el-upload el-upload--picture-card" v-show="!uploadBtn">
            <i class="el-icon-plus"></i>
            <input 
                type="file" 
                name="upload" 
                accept="image/jpeg,image/jpg,image/png" 
                class="el-upload__input" 
                title="上传图片"
                @change="handleUpload"
                ref="upload"
                value=""
            >
        </div>
        <!-- 裁剪按钮 -->
        <el-button 
            type="warning" 
            icon="el-icon-check"
            @click="cropper"
            v-show="showCropper"
            class="cropper-btn"
        >裁剪</el-button>
    </div>
</template>
<script>
import VueCropper from 'vue-cropper'

export default {
    name: 'Upload',
    props: ['imgValue'],
    data() {
        return {
            dataUrl: '',
            showImg: false,
            showCropper: false,
            uploadBtn: false
        }
    },
    components: {
        VueCropper
    },
    methods: {
        handleUpload(e) {
            let inputUpload = this.$refs.upload;
            this.file = inputUpload.files[0]
            
            // console.log(this.file);

            // 文件大小不超过300K
            let size = Math.floor(this.file.size / 1024)

            if(size > 300){
                this.$notify.error({
                    title: '错误',
                    message: '文件大小不得超出300K'
                });
                return false;
            }else{
                // inputUpload.val = this.file
                this.imgPreview(this.file)
                this.showCropper = this.uploadBtn = true
            }
        },
        // 图片转成 base64 格式的 url
        imgPreview(file) {
            let that = this
            let reader = new FileReader()
            // 将图片转成 base64 格式
            reader.readAsDataURL(file)
            // 读取成功后的回调
            reader.onloadend = function (){
                that.dataUrl = this.result
                
                // console.log('result', this.result);
            }
        },
        cropper() {
            // this.$refs.cropper.getCropData((data) => {
            //     // do something
            //     console.log(data)  
            // })
            // console.log(this.$refs.cropper);
            this.$refs.cropper.startCrop()
            this.$refs.cropper.getCropData((data) => {
                // do something
                // console.log(data)
                this.dataUrl = data
                this.showCropper = false
                this.showImg = true
            })
            // 主动触发upload, dataUrl为向父组件传递的数据
            this.$emit('upload', this.dataUrl)
        },
        handleDel() {
            this.dataUrl = ''
            this.showImg = this.uploadBtn = false
            this.$refs.upload.value = ''
            // 主动触发 deleteImg 方法，this.dataUrl 为向父组件传递的数据
            this.$emit('deleteImg', this.dataUrl)
        }
    }
}
</script>
<style lang="scss" scoped>
.el-upload--picture-card{
    position: relative;
}
.el-upload__input{
    opacity: 0;
    position: absolute;
    top: 0;
    left: 0;
    display: block;
    width: 148px;
    height: 148px;
    cursor: pointer;
}
[class*=" el-icon-"],
[class^=el-icon-]{
    font-family: element-icons!important;
    speak: none;
    font-style: normal;
    font-weight: 400;
    font-variant: normal;
    text-transform: none;
    line-height: 1;
    vertical-align: baseline;
    display: inline-block;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}
.preview-wrap{
    position: relative;
    width: 128px;
    height: 128px;
    border-radius: 6px;
    overflow: hidden;
    .preview{
        width: 128px;
        height: 128px;
    }
    .el-upload-list__item-actions{
        width: 100%;
        height: 100%;
        position: absolute;
        left: 0;
        top: 0;
        text-align: center;
        background-color: rgba(0, 0, 0, .5);
        transition: opacity .3s;
        opacity: 0;

        &:hover{
            opacity: 1;
            .el-upload-list__item-delete{
                display: inline-block;
                color: #fff;
            }
        }
        .el-upload-list__item-delete{
            cursor: pointer;
            position: static;
            text-align: center;
            font-size: 20px;
            margin-top: 50px;
        }
        .el-icon-delete{
            &::before{
                content: "\E612";
            }
        }
    }
}
.cropper-wrap{
    width: 400px;
    height: 400px;
    display: inline-block;
}
.cropper-btn{
    display: inline-block;
    margin-top: -30px;
    position: relative;
    top: -14px;
}
</style>
