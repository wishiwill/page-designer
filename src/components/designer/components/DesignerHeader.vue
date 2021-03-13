<template>
    <div class="designer-header-container">
        <el-button plain @click="preview" size="medium">预览</el-button>
        <el-button type="primary" @click="submit" size="medium">提交</el-button>
        <PreviewPage :pageInfo="pageInfo" :components="components" v-model="showPreview"></PreviewPage>
    </div>
</template>

<script>
    import PreviewPage from './components/PreviewPage';
    import {mapMutations, mapState} from 'vuex';

    export default {
        name: 'DesignerHeader',
        components: {
            PreviewPage
        },
        data() {
            return {
                previewDialog: false,
                showPreview: false
            };
        },
        methods: {
            ...mapMutations(['setScrollId', 'setCurrent', 'setValidateKey']),
            preview() {
                this.validateAll().then(() => {
                    this.showPreview = true;
                });
            },
            closePreview() {
                this.showPreview = false;
            },
            submit() {
                this.validateAll().then(() => {
                    let components = this.$_.cloneDeep(this.components);
                    components.forEach(value => {
                        if (value.component === 'Goods') {
                            // 删除goods详情
                            value.list.forEach(item => {
                                item.data = null;
                            });
                        }
                        if (value.component === 'RichText') {
                            //  转base64
                            value.content = value.content.toBase64();
                        }
                    });
                    this.pageInfo.components = components;
                    console.log(JSON.stringify(this.pageInfo));
                    this.$message.success('请在控制台查看数据');
                });
            },
            validateAll() {
                // 先校验pageInfo
                return this.validateForm(this.pageInfo, this.formRules.Page).then(() => {
                    // 递归校验再校验components
                    if (this.components.length) {
                        return this.recursiveValidate();
                    } else {
                        this.$message.error('请添加组件');
                        throw new Error();
                    }
                }).catch(({errors, fields}) => {
                    if (errors?.length) {
                        this.setCurrent(this.pageInfo);
                        this.setScrollId('pageInfo');
                        this.$message.error(errors[0].message);
                        this.setValidateKey();
                    }
                    throw new Error();
                });
            },
            // 递归校验
            recursiveValidate(index = 0) {
                let component = this.components[index];
                return this.validateForm(component, this.formRules[component.component]).then(() => {
                    if (index < this.components.length - 1) {
                        return this.recursiveValidate(index + 1);
                    }
                }).catch(({errors, fields}) => {
                    if (errors?.length) {
                        this.setCurrent(component);
                        this.setScrollId(component.id);
                        this.$message.error(errors[0].message);
                        this.setValidateKey();
                    }
                    throw new Error();
                });
            }
        },
        computed: {
            ...mapState(['shopInfo']),
            ...mapState(['components', 'status', 'pageInfo', 'storeId'])
        }
    };
</script>

<style lang="scss" scoped>
    .designer-header-container {
        height: 100%;
        display: flex;
        justify-content: flex-end;
        align-items: center;
    }
</style>