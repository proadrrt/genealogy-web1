<template>
    <div>
        <Button type="primary" @click="toEdit(0)">添加文章</Button>
        <Table border :columns="columns" :data="list" style="margin:16px 0;"></Table>
        <Page :total="total" @on-change="chgPage" :page-size="8"/>
        <Drawer :mask-closable="false" :title="formData.id ? '修改':'添加'" :width="80" v-model="isedit">
            <Form :model="formData" :label-width="80">
                <FormItem label="标题">
                    <Input :maxlength="45" v-model="formData.newsTitle" placeholder="标题" @keyup.enter.native="toSubmit"/>
                </FormItem>
                <FormItem label="预览图">
                    <Upload class="upload" :action="api.admin.base + api.admin.upload_img" name="file" :show-upload-list="false" :on-success="handleSuccess" :format="['jpg','jpeg','png']">
                        <Button type="dashed">
                            <div class="img" :style="api.imgBG(filePath)" v-if="filePath"/>
                            <div class="img" :style="api.imgBG(formData.newsUploadFileList[0].filePath)" v-else-if="formData.newsUploadFileList.length"></div>
                            <Icon type="ios-camera" size="40" color="#ccc" v-else></Icon>
                        </Button>
                    </Upload>
                </FormItem>
                <FormItem label="正文">
                    <editor ref="editor" @on-change="handleChange"/>
                </FormItem>
                <FormItem label="浏览数" v-if="formData.id">
                    <Input v-model="formData.visitNum" placeholder="浏览数" @keyup.enter.native="toSubmit"/>
                </FormItem>
                <FormItem label>
                    <Button type="primary" @click="toSubmit" style="margin-right:16px;">提交</Button>
                    <Button @click="toDrft" v-if="formData.status != 1" style="margin-right:16px;">存为草稿</Button>
                    <Button @click="isedit = false">关闭</Button>
                </FormItem>
            </Form>
        </Drawer>
    </div>
</template>

<script>
import Editor from "_c/editor";
export default {
    components: {
        Editor
    },
    data() {
        return {
            isedit: false,
            list: [],
            total: 0,
            page: 1,
            formData: {
                id: "",
                newsTitle: "",
                newsText: "",
                newsUploadFileList: []
            },
            fileName: "",
            filePath: "",
            columns: [
                { title: "ID", minWidth: 80, key: "id" },
                {
                    title: " ",
                    width: 64,
                    align: "center",
                    key: "status",
                    render: (h, e) => {
                        if (e.row.status != 2) {
                            return null;
                        }
                        return h("div", '草稿');
                    }
                },
                { title: "标题", key: "newsTitle", minWidth: 480 },
                { title: "浏览数", key: "visitNum", minWidth: 80 },
                { title: "日期", key: "datetime", width: 160 },
                {
                    title: "操作",
                    key: "action",
                    width: 160,
                    align: "center",
                    render: (h, params) => {
                        return h("div", [
                            h("Button", {
                                props: {
                                    type: "primary",
                                    size: "small",
                                    icon: "ios-create"
                                },
                                style: {
                                    marginRight: "5px"
                                },
                                on: {
                                    click: () => {
                                        this.toEdit(this.list[params.index].id);
                                    }
                                }
                            }),
                            h("Button", {
                                props: {
                                    type: "error",
                                    size: "small",
                                    icon: "md-trash"
                                },
                                on: {
                                    click: () => {
                                        this.remove(params.index);
                                    }
                                }
                            })
                        ]);
                    }
                }
            ]
        };
    },
    computed: {
        admin() {
            return this.$store.state.admin;
        }
    },
    watch: {
        url: function(curVal, oldVal) {
            if (curVal != oldVal) {
                this.getList();
            }
        }
    },
    mounted: function() {
        this.getList();
    },
    methods: {
        getList(e) {
            this.api
                .get(this.url, {
                    pageNo: this.page
                })
                .then(res => {
                    if (res.code == 200) {
                        let list = res.data.records;
                        list.forEach(v => {
                            v.datetime = this.dayjs(v.updateTime).format(
                                "YYYY-MM-DD HH:mm:ss"
                            );
                        });
                        this.list = list;
                        this.total = res.data.total;
                    } else {
                        this.list = [];
                    }
                });
        },
        chgPage(e) {
            this.page = e;
            this.getList();
        },
        handleChange(html, text) {
            this.formData.newsText = html;
        },
        toEdit(e) {
            this.fileName = "";
            this.filePath = "";
            this.$refs.editor.setHtml("");
            if (!e) {
                this.formData = {
                    id: "",
                    newsTitle: "",
                    newsText: "",
                    newsUploadFileList: []
                };
                this.isedit = true;
            } else {
                this.api
                    .get(
                        this.api.adminUrl("culture_news_info", this.admin.step),
                        {
                            id: e
                        }
                    )
                    .then(res => {
                        if (res.code == 200) {
                            this.formData = res.data;
                            this.$refs.editor.setHtml(this.formData.newsText);
                            this.isedit = true;
                        } else {
                            this.$Message.error("发生错误");
                        }
                    });
            }
        },
        remove(index) {
            this.$Modal.confirm({
                title: "提示",
                content: "确定删除这个文章？",
                onOk: () => {
                    this.api
                        .get(
                            this.api.adminUrl(
                                "culture_news_del",
                                this.admin.step
                            ),
                            {
                                id: this.list[index].id
                            }
                        )
                        .then(res => {
                            this.list.splice(index, 1);
                            this.getList();
                        });
                }
            });
        },
        handleSuccess(res, file) {
            if (res.code == 200) {
                console.log(res);
                this.fileName = res.data.file_name;
                this.filePath = res.data.file_path;
            }
        },
        toSubmit() {
            if (!this.formData.newsTitle) {
                this.$Message.error("未输入标题");
                return;
            }
            if (!this.formData.newsText) {
                this.$Message.error("未输入正文");
                return;
            }
            let data = {
                showId: this.type,
                newsTitle: this.formData.newsTitle,
                newsText: this.formData.newsText,
                visitNum: this.formData.visitNum ? this.formData.visitNum : 0,
                fileName: this.fileName,
                filePath: this.filePath
            };
            if (this.formData.id) {
                data.id = this.formData.id;
            }
            this.api
                .post(
                    this.api.adminUrl("culture_news_edit", this.admin.step),
                    data
                )
                .then(res => {
                    if (res.code === 200) {
                        if (data.id) {
                            this.$Message.success("修改成功");
                        } else {
                            this.$Message.success("添加成功");
                        }
                        this.getList();
                        this.isedit = false;
                    } else {
                        this.$Message.warning(res.msg);
                        return;
                    }
                });
        },
        toDrft() {
            if (!this.formData.newsTitle) {
                this.$Message.error("未输入标题");
                return;
            }
            if (!this.formData.newsText) {
                this.$Message.error("未输入正文");
                return;
            }
            let data = {
                showId: this.type,
                newsTitle: this.formData.newsTitle,
                newsText: this.formData.newsText,
                visitNum: this.formData.visitNum ? this.formData.visitNum : 0,
                fileName: this.fileName,
                filePath: this.filePath
            };
            if (this.formData.id) {
                data.id = this.formData.id;
            }
            this.api
                .post(
                    this.api.adminUrl("culture_news_drft", this.admin.step),
                    data
                )
                .then(res => {
                    if (res.code === 200) {
                        if (data.id) {
                            this.$Message.success("修改成功");
                        } else {
                            this.$Message.success("添加成功");
                        }
                        this.getList();
                        this.isedit = false;
                    } else {
                        this.$Message.warning(res.msg);
                        return;
                    }
                });
        }
    },
    props: ["url", "menu", "type"]
};
</script>
<style lang="scss" scoped>
.upload {
    button {
        width: 162px;
        height: 122px;
        padding: 0;
        .img {
            width: 160px;
            height: 120px;
            padding: 0;
            background: no-repeat center / cover;
        }
    }
}
</style>
