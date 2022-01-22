<template>
    <el-image ref="images" fit="cover" style="width: 100%;height: 100%" :src="imgRource"></el-image>
</template>

<script>
    export default {
        props: {
            src: String,
            range: {
                type: Number,
                default: 60
            },
            imgListBig:{
                type: Array
            },
            zIndex: {
                type: Number,
            },
            TYPE: {
                type: String
            }
        },
        computed: {
            imgRource() {
                if(this.$props.TYPE == 1){
                    return `${this.$props.src}?x-oss-process=image/quality,q_${this.$props.range}`;
                }
                return this.$props.src;

            }
        },
        methods:{
        }
    }
</script>

<style scoped>

</style>