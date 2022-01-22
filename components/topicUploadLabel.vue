<template>
    <div>
        <el-input
                class="input-new-tag"
                v-if="inputVisible"
                v-model="inputValue"
                ref="saveTagInput"
                size="mini"
                maxlength="10"
                show-word-limit
                @keyup.enter.native="handleInputConfirm"
                @blur="handleInputConfirm"
        >
        </el-input>
        <el-button v-else-if="!inputVisible && dynamicTagslabel.length<5" class="button-new-tag" size="mini"  @click="showInput">请输入标签</el-button>
        <el-tag
                :key="tagindex"
                v-for="(tag,tagindex) in dynamicTagslabel"
                closable
                :disable-transitions="true"
                @close="handleClose(tag)"
                size="small"
        >
            {{tag}}
        </el-tag>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                inputVisible: false,
                inputValue: '',
                loadProgress:0,
                status:''
            };
        },
        methods: {
            handleClose(tag) {
                this.dynamicTagslabel.splice(this.dynamicTagslabel.indexOf(tag), 1);
            },
            showInput() {
                this.inputVisible = true;
                this.$nextTick(_ => {
                    this.$refs.saveTagInput.$refs.input.focus();
                });
            },
            handleInputConfirm() {
                let inputValue = this.inputValue;
                if (inputValue) {
                    this.dynamicTagslabel.push(inputValue);
                }
                this.inputVisible = false;
                this.inputValue = '';
            }
        },
        watch:{
            dynamicTagslabel(){
                this.$emit('listentoLabel',this.dynamicTagslabel,this.index);
            },
        },
        props:['index','dynamicTagslabel']
    }
</script>

<style scoped>
    .el-button--mini, .el-button--mini.is-round {
        width: 160px;
        margin-bottom: 3px;
    }
    .input-new-tag{
        width: 160px;
        margin-bottom: 3px;
    }
</style>