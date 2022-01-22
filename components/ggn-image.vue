<template>
    <el-image class="ggn-img" z-index=9999 fit="cover" style="width: 100%;" :src="imgRource" alt=""  :preview-src-list="getSrcList()"></el-image>
</template>

<script>
    export default {
        props: {
            src: String,
            range: {
                type: Number,
                default: 60
            },
            width: {
                type: Number,
                default: 100
            },
            height: {
                type: Number,
                default: 100
            },
            imgListBig: {
                type:Array
            },
            zIndex:{
                type:Number,
            },
            TYPE:{
                type:String
            },
            TYPE:{
                type:String
            }
        },
        computed: {
            imgRource() {
                return `${this.$props.src}?x-oss-process=image/quality,q_${this.$props.range}/resize,m_mfit,h_${this.$props.height},w_${this.$props.width}`;
            }
        },
        methods:{
            getSrcList(){
                if(this.$props.TYPE == '1'){
                    const _imgListBig = this.$props.imgListBig.map(e =>{
                        return e.url;
                    })
                    return _imgListBig && _imgListBig.slice(this.$props.zIndex).concat(_imgListBig.slice(0,this.$props.zIndex));
                }
                if(this.$props.TYPE == '2'){
                    const _imgListBig = this.$props.imgListBig.map(e =>{
                        return e.attachUrl;
                    })
                    return _imgListBig && _imgListBig.slice(this.$props.zIndex).concat(_imgListBig.slice(0,this.$props.zIndex));
                }
                return this.$props.imgListBig && this.$props.imgListBig.slice(this.$props.zIndex).concat(this.$props.imgListBig.slice(0,this.$props.zIndex));
            }
        }
    }
</script>
<style lang="less" scoped>
  .ggn-img{
    /deep/.el-icon-circle-close {
      color: white;
    }
    .el-table{
      color: #fff;
    }
  }
</style>