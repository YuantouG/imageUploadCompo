<template>
    <div>
       <template v-if="progressFlag && progressShow">
            <el-progress type="circle" :percentage="loadProgress"></el-progress>
           <span @click="pauseUp" style="position: absolute;top: 4px;left: 135px;font-size: 12px;cursor: pointer;">❌</span>
       </template>
        <template v-else>
            <label class="el-upload-list__item-status-label"><i class="el-icon-upload-success el-icon-check"></i></label>
            <i class="el-icon-close"></i>
            <i class="el-icon-close-tip">按 delete 键可删除</i>
            <span class="el-upload-list__item-actions">
                <span class="el-upload-list__item-preview" @click="onPreview()">
                <i class="el-icon-zoom-in"></i>
                </span>
                <span class="el-upload-list__item-delete" @click="deleteImage(item)">
                    <i class="el-icon-delete"></i>
                </span>
            </span>
        </template>
        <el-image-viewer v-if="showViewer" :on-close="closeViewer" :url-list="viewerImgList"/>
    </div>
</template>

<script>
    import ElImageViewer from "element-ui/packages/image/src/image-viewer.vue"
    export default {
        data(){
            return{
                loadProgress:0,
                progressFlag:true,
                checkpointel:{},
                showViewer:false,
                viewerImgList:[],
            }
        },
        components:{
            ElImageViewer
        },
        methods:{
            onPreview() {
                    this.showViewer = true;
                    let tempImgList = this.imgListBig.map(e =>{
                        return e.relurl+'?x-oss-process=image/auto-orient,1/resize,m_fixed,w_1080,h_0/quality,q_60/watermark,image_R0dOL1dBVEVSTUFSSy9hZGNhdHRsZXdhdGVyLnBuZz94LW9zcy1wcm9jZXNzPWltYWdlL3Jlc2l6ZSxQXzIw,x_10,y_10';
                    })
                    let temp = [];
                    for (let i = 0; i < this.index; i++) {
                        //shift() 方法用于把数组的第一个元素从其中删除，并返回第一个元素的值。
                        temp.push(tempImgList.shift());
                    }
                    this.viewerImgList = tempImgList.concat(temp);//将当前图片调整成点击缩略图的那张图片
            },
            // 关闭查看器
            closeViewer() {
                this.showViewer = false
            },
            //删除图片
            deleteImage(){
                this.$emit('deleteImage',this.item);
            },
            //上传中-取消上传
            pauseUp(){
                this.$emit('stopuload',this.checkpointel,this.item);
            }
        },
        watch:{
            //监听进度条，上传完成时，使图片下面的标签显示
            loadProgress(){
               if(this.loadProgress == 100){
                   this.$emit('listentoprog',true,this.index);
               }
            },
        },
        props:['records','item','index','progressShow','imgListBig'],
    }
</script>

<style scoped>
</style>