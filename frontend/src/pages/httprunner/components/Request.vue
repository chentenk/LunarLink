<template>
    <div>
        <div style="margin-left: 200px;">
            <el-radio-group v-model="dataType">
                <el-radio
                    v-for="item of dataOptions"
                    :label="item.label"
                    :key="item.value"
                    >{{ item.value }}</el-radio
                >
            </el-radio-group>
        </div>
        <div>
            <el-dialog
                title="批量编辑"
                :visible.sync="showDialog"
                :close-on-click-modal="false"
                width="45%"
            >
                <div style="margin-bottom: 8px;">
                    <span style="color: #7d858e;">
                        格式:&nbsp;<span>参数名:</span><span>参数值</span>
                    </span>
                </div>
                <el-input
                    type="textarea"
                    :autosize="{ minRows: 10, maxRows: 16 }"
                    v-model="textareaData"
                ></el-input>
                <div style="margin-top: 8px; color: #7d858e;">
                    字段之间以英文冒号( : )分隔，多条记录以换行分隔
                </div>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="showDialog = false">取 消</el-button>
                    <el-button type="primary" @click="handleSubmit"
                        >确 定</el-button
                    >
                </span>
            </el-dialog>
        </div>
        <div style="margin-top: 5px">
            <el-table
                higlight-current-row
                stripe
                :cell-style="{ paddingTop: '4px', paddingBottom: '4px' }"
                :height="height"
                :data="dataType === 'data' ? formData : paramsData"
                style="width: 100%;"
                @cell-mouse-enter="cellMouseEnter"
                @cell-mouse-leave="cellMouseLeave"
                v-show="dataType !== 'json'"
            >
                <!--Request Params || Data-->
                <el-table-column label="参数名" width="250">
                    <template slot-scope="scope">
                        <el-input
                            clearable
                            v-model.trim="scope.row.key"
                            placeholder="Key"
                        ></el-input>
                    </template>
                </el-table-column>
                <!--Request 表单-->
                <el-table-column
                    label="类型"
                    width="120"
                    v-if="dataType === 'data'"
                >
                    <template slot-scope="scope">
                        <el-select v-model="scope.row.type">
                            <el-option
                                v-for="item in dataTypeOptions"
                                :key="item.key"
                                :label="item.label"
                                :value="item.value"
                            ></el-option>
                        </el-select>
                    </template>
                </el-table-column>

                <el-table-column label="参数值" width="350">
                    <template slot-scope="scope">
                        <el-input
                            v-show="scope.row.type !== 5"
                            clearable
                            v-model.trim="scope.row.value"
                            placeholder="Value"
                        ></el-input>

                        <!-- TODO: 屏蔽文件上传功能 -->
                        <!--                        <el-row v-show="scope.row.type === 5">-->
                        <!--                            <el-col :span="7">-->
                        <!--                                <el-upload-->
                        <!--                                    :show-file-list="false"-->
                        <!--                                    :action="uploadFile(scope.row)"-->
                        <!--                                    :limit="1"-->
                        <!--                                    type="small"-->
                        <!--                                    :file-list="fileList"-->
                        <!--                                    :on-error="uploadError"-->
                        <!--                                    :on-success="uploadSuccess"-->
                        <!--                                >-->
                        <!--                                    <el-button-->
                        <!--                                        size="small"-->
                        <!--                                        type="primary"-->
                        <!--                                        @click="currentIndex = scope.$index"-->
                        <!--                                        >选择文件</el-button-->
                        <!--                                    >-->
                        <!--                                </el-upload>-->
                        <!--                            </el-col>-->
                        <!--                            <el-col :span="12">-->
                        <!--                                <el-badge-->
                        <!--                                    :value="scope.row.size"-->
                        <!--                                    style="margin-top: 8px"-->
                        <!--                                >-->
                        <!--                                    <i-->
                        <!--                                        class="el-icon-document"-->
                        <!--                                        v-text="scope.row.value"-->
                        <!--                                    ></i>-->
                        <!--                                </el-badge>-->
                        <!--                            </el-col>-->
                        <!--                        </el-row>-->
                    </template>
                </el-table-column>

                <el-table-column label="描述" width="400">
                    <template slot-scope="scope">
                        <el-input
                            clearable
                            v-model.trim="scope.row.desc"
                            placeholder="参数简要描述"
                        ></el-input>
                    </template>
                </el-table-column>

                <el-table-column>
                    <template slot="header" slot-scope="scope">
                        <div
                            v-show="isHoveringHeader || isHoveringRow"
                            style="cursor: pointer; font-weight: bold; color: #409EFF; font-size: 14px;"
                            @click="handleBulkEdit"
                        >
                            <i class="el-icon-edit"></i>
                            <span>批量编辑</span>
                        </div>
                    </template>
                    <template slot-scope="scope">
                        <el-row v-show="scope.row === currentRow">
                            <el-button
                                icon="el-icon-circle-plus-outline"
                                size="mini"
                                type="info"
                                @click="handleEdit(scope.$index, scope.row)"
                            ></el-button>
                            <el-button
                                icon="el-icon-document-copy"
                                size="mini"
                                type="info"
                                @click="handleCopy(scope.$index, scope.row)"
                            ></el-button>
                            <el-button
                                icon="el-icon-delete"
                                size="mini"
                                type="danger"
                                v-show="scope.$index !== 0"
                                @click="handleDelete(scope.$index, scope.row)"
                            ></el-button>
                        </el-row>
                    </template>
                </el-table-column>
            </el-table>
            <!--Request JSON-->
            <v-jsoneditor
                v-model="editorJsonData"
                v-show="dataType === 'json'"
                :key="uniqueKey"
                :height="height"
                :options="options"
                :plus="true"
                ref="requestEditor"
            ></v-jsoneditor>
        </div>
    </div>
</template>

<script>
import bus from "@/util/bus";
import { isEqual } from "lodash";

export default {
    name: "Request",
    props: {
        save: Boolean,
        request: {
            required: false
        },
        resetDataType: {
            type: Boolean,
            default: false
        }
    },
    data() {
        let self = this;
        return {
            options: {
                // 从树形模式切换到代码模式时，该函数会将JSON编辑器的所有目录全部展开
                onModeChange(newMode, oldMode) {
                    if (newMode === "tree") {
                        self.$refs.requestEditor.editor.expandAll();
                    }
                },
                // 当使用alt键单击tree目录时，会从当前目录中提取出JSON Path并插入到一个名为extract的数组中
                onEvent(node, event) {
                    if (event.type === "click" && event.altKey) {
                        // 左键点击 + alt 提取请求参数jsonpath插入到extract中
                        let arr = node.path;
                        arr.unshift("request.body");
                        let jsonPath = arr.join(".");
                        self.notifyCopyRequest(jsonPath, "Extract插入");
                        const extractObj = {
                            key: arr[arr.length - 1],
                            value: jsonPath,
                            desc: ""
                        };
                        bus.$emit("extractRequest", extractObj);
                    }
                },
                mode: "code",
                modes: ["code", "tree"] // allowed modes
            },
            editorJsonData: {},
            fileList: [],
            currentIndex: 0,
            currentRow: "",
            originalData: {
                formData: [
                    {
                        key: "",
                        value: "",
                        type: 1,
                        desc: ""
                    }
                ],
                paramsData: [
                    {
                        key: "",
                        value: "",
                        type: 1,
                        desc: ""
                    }
                ],
                editorJsonData: {}
            },
            formData: [
                {
                    key: "",
                    value: "",
                    type: 1,
                    desc: ""
                }
            ],
            paramsData: [
                {
                    key: "",
                    value: "",
                    type: 1, // 任意值，类型为number就行, 实际上不会用到
                    desc: ""
                }
            ],
            dataTypeOptions: [
                {
                    label: "String",
                    value: 1
                },
                {
                    label: "Integer",
                    value: 2
                },
                {
                    label: "Float",
                    value: 3
                },
                {
                    label: "Boolean",
                    value: 4
                }
                // {
                //     label: "File",
                //     value: 5
                // }
            ],
            dataOptions: [
                {
                    label: "data",
                    value: "表单"
                },
                {
                    label: "json",
                    value: "json"
                },
                {
                    label: "params",
                    value: "params"
                }
            ],
            dataType: "data",
            timeStamp: "",
            showDialog: false,
            isHoveringHeader: false,
            isHoveringRow: false,
            textareaData: "",
            uniqueKey: Date.now()
        };
    },
    computed: {
        height() {
            return (window.screen.height - 464).toString() + "px";
        }
    },
    mounted() {
        this.setOriginalData();

        this.editorJsonData = this.parseJson();

        const table = this.$el.querySelector(".el-table__header-wrapper");
        table.addEventListener("mouseenter", () => {
            this.isHoveringHeader = true;
        });
        table.addEventListener("mouseleave", () => {
            this.isHoveringHeader = false;
        });
    },
    watch: {
        save() {
            // 保存 另存为 发送都会触发
            this.$emit(
                "request",
                {
                    form: this.parseForm(),
                    json: this.editorJsonData,
                    params: this.parseParams(),
                    files: this.parseFile()
                },
                {
                    // 编辑用例步骤会用到
                    data: this.formData,
                    params: this.paramsData,
                    json_data: this.editorJsonData
                }
            );
        },
        request: {
            deep: true,
            handler(newVal) {
                if (newVal.length !== 0) {
                    this.uniqueKey = Date.now();
                    this.formData = JSON.parse(JSON.stringify(newVal.data));
                    this.editorJsonData = JSON.parse(
                        JSON.stringify(this.parseJson())
                    );
                    this.paramsData = JSON.parse(JSON.stringify(newVal.params));
                    this.setOriginalData();
                }
            }
        },
        resetDataType() {
            this.dataType = "data";
        },
        formData: {
            deep: true,
            handler() {
                this.checkDataChanges();
            }
        },
        paramsData: {
            deep: true,
            handler() {
                this.checkDataChanges();
            }
        },
        editorJsonData: {
            deep: true,
            handler() {
                this.checkDataChanges();
            }
        }
    },
    methods: {
        setOriginalData() {
            if (this.request) {
                this.originalData = {
                    formData: this.request.data,
                    paramsData: this.request.params,
                    editorJsonData: this.parseJson()
                };
            }
        },
        checkDataChanges() {
            const currentData = {
                formData: this.formData,
                paramsData: this.paramsData,
                editorJsonData: this.editorJsonData
            };
            const hasChanged = !isEqual(this.originalData, currentData);
            this.$emit("dataChanged", hasChanged);
        },
        handleSubmit() {
            this.showDialog = false;
            const lines = this.textareaData.split("\n");
            let data = lines
                .filter(item => item.trim() !== "") // 过滤掉空的行
                .map(item => {
                    const colonIndex = item.indexOf(":"); // 找到第一个冒号的位置
                    if (colonIndex === -1) {
                        // 如果没有找到冒号，跳过这一行
                        return null;
                    }
                    const key = item.substring(0, colonIndex).trim(); // 提取key
                    const value = item.substring(colonIndex + 1).trim(); // 提取value
                    return {
                        key,
                        value,
                        type: 1,
                        desc: ""
                    };
                })
                .filter(item => item !== null); // 过滤掉无效的行

            if (data.length === 0) {
                // 如果data为空，添加一行默认数据
                data.push({
                    key: "",
                    value: "",
                    type: 1,
                    desc: ""
                });
            }

            if (this.dataType === "data") {
                this.formData = data;
            } else {
                this.paramsData = data;
            }
        },
        handleBulkEdit() {
            this.showDialog = true;
            const data =
                this.dataType === "data" ? this.formData : this.paramsData;
            this.textareaData = data
                .filter(item => item.key || item.value) // 确保key或value存在
                .map(item => {
                    const flag = ":";
                    return `${item.key}${flag}${item.value}`;
                })
                .join("\n");
        },
        uploadSuccess(response, file, fileList) {
            let size = file.size;
            if (size >= 1048576) {
                size = (size / 1048576).toFixed(2).toString() + "MB";
            } else if (size > 1024) {
                size = (size / 1024).toFixed(2).toString() + "KB";
            } else {
                size = size.toString() + "Byte";
            }
            this.formData[this.currentIndex]["value"] = file.name;
            this.formData[this.currentIndex]["size"] = size;
            this.fileList = [];
            if (!response.success) {
                this.$message.error(file.name + response.msg);
            }
        },
        uploadFile(row) {
            return this.$api.uploadFile();
        },
        uploadError(err, file, fileList) {
            if (err.status === 401) {
                this.$message.error("登录已过期，请重新登录");
                this.$router.replace({
                    name: " Login"
                });
            } else {
                this.$message.error("文件上传失败啦, 请重试!");
            }
        },
        cellMouseEnter(row) {
            this.currentRow = row;
            this.isHoveringRow = true;
        },
        cellMouseLeave() {
            this.currentRow = "";
            this.isHoveringRow = false;
        },
        handleEdit(index, row) {
            const data =
                this.dataType === "data" ? this.formData : this.paramsData;
            data.push({
                key: "",
                value: "",
                type: 1,
                desc: ""
            });
        },
        handleCopy(index, row) {
            const data =
                this.dataType === "data" ? this.formData : this.paramsData;
            data.splice(index + 1, 0, {
                key: row.key,
                value: row.value,
                type: row.type,
                desc: row.desc
            });
        },
        handleDelete(index, row) {
            const data =
                this.dataType === "data" ? this.formData : this.paramsData;
            data.splice(index, 1);
        },
        notifyCopyRequest(jsonPath, title) {
            this.$notify.success({
                title: title,
                message: jsonPath,
                duration: 2000
            });
        },
        parseJson() {
            const jsonStr = this.request.json_data;
            if (typeof jsonStr === "object") {
                return jsonStr;
            }

            if (typeof jsonStr === "string" && jsonStr.trim() !== "") {
                try {
                    return JSON.parse(jsonStr);
                } catch (err) {
                    this.$notify.error({
                        title: "JSON解析错误",
                        message: "不是标准的json数据格式",
                        duration: 2000
                    });
                    // 可以在这里添加额外的错误处理逻辑
                }
            }

            return {}; // 如果 jsonStr 不是对象或有效的 JSON 字符串，返回空对象
        },
        // 文件格式化
        parseFile() {
            let files = {
                files: {},
                desc: {}
            };

            for (let content of this.formData) {
                // 文件
                if (content["key"] !== "" && content["type"] === 5) {
                    files["files"][content["key"]] = content["value"];
                    files["desc"][content["key"]] = content["desc"];
                }
            }
            return files;
        },
        // 表单格式化
        parseForm() {
            let form = {
                data: {},
                desc: {}
            };
            for (let content of this.formData) {
                // file 不处理
                if (content["key"] !== "" && content["type"] !== 5) {
                    const value = this.parseType(
                        content["type"],
                        content["value"]
                    );

                    if (value === "exception") {
                        continue;
                    }

                    form.data[content["key"]] = value;
                    form.desc[content["key"]] = content["desc"];
                }
            }
            return form;
        },
        parseParams() {
            let params = {
                params: {},
                desc: {}
            };
            for (let content of this.paramsData) {
                if (content["key"] !== "") {
                    params.params[content["key"]] = content["value"];
                    params.desc[content["key"]] = content["desc"];
                }
            }
            return params;
        },
        // 根据类型转换
        parseType(type, value) {
            let tempValue;
            const msg =
                value +
                " => " +
                this.dataTypeOptions[type - 1].label +
                " 转换异常, 该数据自动剔除";
            switch (type) {
                case 1:
                    tempValue = value;
                    break;
                case 2:
                    tempValue = parseInt(value);
                    break;
                case 3:
                    tempValue = parseFloat(value);
                    break;
                case 4:
                    if (value === "False" || value === "True") {
                        let bool = {
                            True: true,
                            False: false
                        };
                        tempValue = bool[value];
                    } else {
                        this.$notify.error({
                            title: "类型转换错误",
                            message: msg,
                            duration: 2000
                        });
                        return "exception";
                    }
                    break;
                case 5:
                case 6:
                    try {
                        tempValue = JSON.parse(value);
                    } catch (err) {
                        // 包含$是引用类型, 可以任意类型
                        if (value.indexOf("$") !== -1) {
                            tempValue = value;
                        } else {
                            tempValue = false;
                        }
                    }
                    break;
            }
            if (tempValue !== 0 && !tempValue && type !== 4 && type !== 1) {
                this.$notify.error({
                    title: "类型转换错误",
                    message: msg,
                    duration: 2000
                });
                return "exception";
            }
            return tempValue;
        }
    }
};
</script>

<style>
.el-dialog__header {
    padding: 20px 20px 0;
}
</style>
