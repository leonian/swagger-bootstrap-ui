<template>
  <div class="document">
    <a-row>
      <a-row class="knife4j-api-title">
        <a-col :span="20">
          <span v-if="api.deprecated" class="knife4j-menu-api-deprecated">
            {{ api.summary }}
          </span>
          <span v-else>
            {{ api.summary }}
          </span>
        </a-col>
        <a-col
          :span="2"
          :id="'btnCopyMarkdown' + api.id"
          class="knife4j-api-copy-address"
          v-html="$t('doc.copy')"
          >复制文档</a-col
        >
        <a-col
          :span="2"
          :id="'btnCopyAddress' + api.id"
          class="knife4j-api-copy-address"
          v-html="$t('doc.copyHash')"
          >复制地址</a-col
        >
      </a-row>
      <a-row :class="'knife4j-api-' + api.methodType.toLowerCase()">
        <div class="knife4j-api-summary">
          <span class="knife4j-api-summary-method">{{ api.methodType }}</span>
          <span class="knife4j-api-summary-path">{{ api.showUrl }}</span>
        </div>
      </a-row>
      <a-row class="knife4j-api-row">
        <a-col :span="12">
          <a-row>
            <a-col class="api-basic-title" :span="6" v-html="$t('doc.produces')"
              >请求数据类型</a-col
            >
            {{ api.consumes }}
          </a-row>
        </a-col>
        <a-col :span="12">
          <a-row>
            <a-col class="api-basic-title" :span="6" v-html="$t('doc.consumes')"
              >响应数据类型</a-col
            >
            {{ api.produces }}
          </a-row>
        </a-col>
      </a-row>
    </a-row>
    <!-- <a-divider class="divider" /> -->
    <!--接口描述-->
    <div v-if="api.description">
      <div class="api-title" v-html="$t('doc.des')">
        接口描述
      </div>
      <div
        v-if="api.description"
        v-html="api.description"
        class="api-body-desc"
      ></div>
    </div>
    <!--请求示例-->
    <div v-if="api.requestValue">
      <div class="api-title" v-html="$t('doc.requestExample')">
        请求示例
      </div>
      <editor-show :value="api.requestValue"></editor-show>
    </div>
    <div class="api-title" v-html="$t('doc.params')">
      请求参数
    </div>
    <a-table
      :defaultExpandAllRows="expanRows"
      :columns="columns"
      :dataSource="reqParameters"
      rowKey="id"
      size="small"
      :pagination="page"
    >
      <template slot="descriptionValueTemplate" slot-scope="text">
        {{ text }}
      </template>
      <template slot="requireTemplate" slot-scope="text">
        <span v-if="text" style="color:red">{{ text.toLocaleString() }}</span>
        <span v-else>{{ text.toLocaleString() }}</span>
      </template>
      <template slot="typeTemplate" slot-scope="text">
        <span :class="'knife4j-request-' + text">{{ text }}</span>
      </template>

      <template slot="datatypeTemplate" slot-scope="text, record">
        <data-type :text="text" :record="record"></data-type>
      </template>
    </a-table>
    <div class="api-title" v-html="$t('doc.response')">
      响应状态
    </div>
    <a-table
      :defaultExpandAllRows="expanRows"
      :columns="responseStatuscolumns"
      :dataSource="api.responseCodes"
      rowKey="code"
      size="small"
      :pagination="page"
    >
      <template slot="descriptionTemplate" slot-scope="text">
        <div v-html="text"></div>
      </template>
    </a-table>
    <!--响应参数需要判断是否存在多个code-schema的情况-->
    <div v-if="api.multipartResponseSchema">
      <a-tabs @change="multipartTabCodeChanges">
        <a-tab-pane
          v-for="resp in multipCodeDatas"
          :key="resp.code"
          :tab="resp.code"
        >
          <!--判断响应头-->
          <div v-if="resp.responseHeaderParameters">
            <div class="api-title" v-html="$t('doc.responseHeaderParams')">
              响应Header
            </div>
            <a-table
              :defaultExpandAllRows="expanRows"
              :columns="responseHeaderColumns"
              :dataSource="resp.responseHeaderParameters"
              rowKey="id"
              size="small"
              :pagination="page"
            >
            </a-table>
          </div>
          <!--响应参数-->
          <div class="api-title" v-html="$t('doc.responseParams')">
            响应参数
          </div>
          <a-table
            :defaultExpandAllRows="expanRows"
            :columns="responseParametersColumns"
            :dataSource="resp.data"
            rowKey="id"
            size="small"
            :pagination="page"
          >
          </a-table>
          <div class="api-title" v-html="$t('doc.responseExample')">
            响应示例
          </div>
          <a-row :id="'knife4jDocumentShowEditor' + api.id + resp.code">
            <editor-show
              :value="
                resp.responseBasicType ? resp.responseText : resp.responseValue
              "
            ></editor-show>
          </a-row>

          <!-- <editor :value="resp.responseBasicType ? resp.responseText : resp.responseValue" @init="multiResponseSampleEditorInit" lang="json" theme="eclipse" width="100%" :height="editorMultiHeight"></editor> -->
        </a-tab-pane>
      </a-tabs>
    </div>
    <div v-else>
      <!--判断响应头-->
      <div v-if="api.responseHeaderParameters">
        <div class="api-title" v-html="$t('doc.responseHeaderParams')">
          响应Header
        </div>
        <a-table
          :defaultExpandAllRows="expanRows"
          :columns="responseHeaderColumns"
          :dataSource="api.responseHeaderParameters"
          rowKey="id"
          size="small"
          :pagination="page"
        >
        </a-table>
      </div>
      <!--响应参数-->
      <div class="api-title" v-html="$t('doc.responseParams')">
        响应参数
      </div>
      <a-table
        :defaultExpandAllRows="expanRows"
        :columns="responseParametersColumns"
        :dataSource="multipData.data"
        rowKey="id"
        size="small"
        :pagination="page"
      >
      </a-table>
      <div class="api-title" v-html="$t('doc.responseExample')">
        响应示例
      </div>
      <a-row :id="'knife4jDocumentShowEditor' + api.id">
        <editor-show
          :value="
            multipData.responseBasicType
              ? multipData.responseText
              : multipData.responseValue
          "
        ></editor-show>
      </a-row>
    </div>
  </div>
</template>
<script>
import KUtils from "@/core/utils";
import Constants from "@/store/constants";
import DataType from "./DataType";
import markdownSingleText from "@/components/officeDocument/markdownSingleTransform";
import EditorShow from "./EditorShow";
import ClipboardJS from "clipboard";
//请求参数table-header
const requestcolumns = [
  {
    title: "参数名称",
    dataIndex: "name",
    width: "30%"
  },
  {
    title: "参数说明",
    dataIndex: "description",
    width: "25%",
    scopedSlots: { customRender: "descriptionValueTemplate" }
  },
  {
    title: "请求类型",
    dataIndex: "in",
    scopedSlots: { customRender: "typeTemplate" }
  },
  {
    title: "是否必须",
    dataIndex: "require",
    scopedSlots: { customRender: "requireTemplate" }
  },
  {
    title: "数据类型",
    dataIndex: "type",
    scopedSlots: { customRender: "datatypeTemplate" }
  },
  {
    title: "schema",
    dataIndex: "schemaValue",
    width: "15%"
  }
];
//响应状态table-header
const responseStatuscolumns = [
  {
    title: "状态码",
    dataIndex: "code",
    width: "20%"
  },
  {
    title: "说明",
    dataIndex: "description",
    width: "55%",
    scopedSlots: { customRender: "descriptionTemplate" }
  },
  {
    title: "schema",
    dataIndex: "schema"
  }
];
//响应头-header
const responseHeaderColumns = [
  {
    title: "参数名称",
    dataIndex: "name",
    width: "30%"
  },
  {
    title: "参数说明",
    dataIndex: "description",
    width: "55%"
  },
  {
    title: "数据类型",
    dataIndex: "type"
  }
];
const responseParametersColumns = [
  {
    title: "参数名称",
    dataIndex: "name",
    width: "35%"
  },
  {
    title: "参数说明",
    dataIndex: "description",
    width: "40%"
  },
  {
    title: "类型",
    dataIndex: "type"
  },
  {
    title: "schema",
    dataIndex: "schemaValue",
    width: "15%"
  }
];
export default {
  name: "Document",
  components: { editor: require("vue2-ace-editor"), DataType, EditorShow },
  props: {
    api: {
      type: Object,
      required: true
    },
    swaggerInstance: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      content: "<span>Hello</span>",
      columns: requestcolumns,
      responseHeaderColumns: responseHeaderColumns,
      responseStatuscolumns: responseStatuscolumns,
      responseParametersColumns: responseParametersColumns,
      expanRows: true,
      //接收一个响应信息对象,遍历得到树形结构的值
      multipCode: false,
      multipCodeDatas: [],
      multipData: {},
      page: false,
      reqParameters: []
    };
  },
  created() {
    var that = this;
    var key = Constants.globalTreeTableModelParams + this.swaggerInstance.id;
    //根据instance的实例初始化model名称
    var treeTableModel = this.swaggerInstance.refTreeTableModels;
    this.$Knife4jModels.setValue(key, treeTableModel);
    this.initRequestParams();
    this.initResponseCodeParams();
    setTimeout(() => {
      that.showResponseEditFieldDescription();
      that.copyApiAddress();
      that.copyApiMarkdown();
    }, 1500);
  },
  methods: {
    copyApiAddress() {
      var that = this;
      var btnId = "btnCopyAddress" + this.api.id;
      var clipboard = new ClipboardJS("#" + btnId, {
        text() {
          return window.location.href;
        }
      });
      clipboard.on("success", function(e) {
        that.$message.info("复制地址成功");
      });
      clipboard.on("error", function(e) {
        that.$message.info("复制地址失败");
      });
    },
    copyApiMarkdown() {
      var that = this;
      var btnId = "btnCopyMarkdown" + this.api.id;
      var api = {
        ...that.api,
        reqParameters: that.reqParameters,
        multipCodeDatas: that.multipCodeDatas,
        multipData: that.multipData
      };
      //console.log(api);
      var clipboard = new ClipboardJS("#" + btnId, {
        text() {
          return markdownSingleText(api);
        }
      });
      clipboard.on("success", function(e) {
        that.$message.info("复制文档成功");
      });
      clipboard.on("error", function(e) {
        that.$message.info("复制文档失败");
      });
    },
    initRequestParams() {
      var key = Constants.globalTreeTableModelParams + this.swaggerInstance.id;
      var data = [];
      var that = this;
      var apiInfo = this.api;
      if (apiInfo.parameters != null && apiInfo.parameters.length > 0) {
        data = data.concat(apiInfo.parameters);
      }
      if (
        apiInfo.refTreetableparameters != null &&
        apiInfo.refTreetableparameters.length > 0
      ) {
        apiInfo.refTreetableparameters.forEach(function(ref) {
          data = data.concat(ref.params);
        });
      }
      if (data != null) {
        data.sort(function(a, b) {
          return b.require - a.require;
        });
      }
      let reqParameters = [];
      if (data != null && data.length > 0) {
        //console("初始化请求参数----------");
        //console(data);
        data.forEach(function(param) {
          if (param.pid == "-1") {
            param.children = [];
            //判断该参数是否存在schema参数
            if (param.schema) {
              //判断当前缓存是否存在
              var schemaName = param.schemaValue;
              if (KUtils.checkUndefined(schemaName)) {
                // //console("schemaValue--checkUndefined");
                if (that.$Knife4jModels.exists(key, schemaName)) {
                  ////console("存在-不用查找---" + schemaName);
                  ////console(that.$Knife4jModels.instance);
                  var model = that.$Knife4jModels.getByModelName(
                    key,
                    schemaName
                  );
                  if (KUtils.checkUndefined(model)) {
                    var children = model.params;
                    if (KUtils.arrNotEmpty(children)) {
                      children.forEach(function(chd) {
                        var target = that.copyNewParameter(chd);
                        target.pid = param.id;
                        param.children.push(target);
                      });
                    }
                  }
                }
              }
            }
            //针对非空的参数,设置children属性为空
            if (!KUtils.arrNotEmpty(param.children)) {
              param.children = null;
              //从knife4jModels中查询参数
            }
            reqParameters.push(param);
          }
        });
      }
      that.reqParameters = reqParameters;
      //console.log(reqParameters);
    },
    copyNewParameter(source) {
      var tmpc = source.children;
      if (!KUtils.checkUndefined(tmpc)) {
        tmpc = null;
      }
      var target = {
        children: tmpc,
        childrenTypes: source.childrenTypes,
        def: source.def,
        description: source.description,
        enum: source.enum,
        example: source.example,
        id: source.id,
        ignoreFilterName: source.ignoreFilterName,
        in: source.in,
        level: source.level,
        name: source.name,
        parentTypes: source.parentTypes,
        pid: source.pid,
        readOnly: source.readOnly,
        require: source.require,
        schema: source.schema,
        schemaValue: source.schemaValue,
        show: source.show,
        txtValue: source.txtValue,
        type: source.type,
        validateInstance: source.validateInstance,
        validateStatus: source.validateStatus,
        value: source.value
      };
      return target;
    },
    deepTreeTableSchemaModel(param, treeTableModel, rootParam) {
      var that = this;
      ////console(model.name)
      if (KUtils.checkUndefined(param.schemaValue)) {
        var schema = treeTableModel[param.schemaValue];
        if (KUtils.checkUndefined(schema)) {
          rootParam.parentTypes.push(param.schemaValue);
          if (KUtils.arrNotEmpty(schema.params)) {
            schema.params.forEach(function(nmd) {
              //childrenparam需要深拷贝一个对象
              var childrenParam = {
                childrenTypes: nmd.childrenTypes,
                def: nmd.def,
                description: nmd.description,
                enum: nmd.enum,
                example: nmd.example,
                id: nmd.id,
                ignoreFilterName: nmd.ignoreFilterName,
                in: nmd.in,
                level: nmd.level,
                name: nmd.name,
                parentTypes: nmd.parentTypes,
                pid: nmd.pid,
                readOnly: nmd.readOnly,
                require: nmd.require,
                schema: nmd.schema,
                schemaValue: nmd.schemaValue,
                show: nmd.show,
                txtValue: nmd.txtValue,
                type: nmd.type,
                validateInstance: nmd.validateInstance,
                validateStatus: nmd.validateStatus,
                value: nmd.value
              };
              childrenParam.pid = param.id;
              param.children.push(childrenParam);
              if (childrenParam.schema) {
                //存在schema,判断是否出现过
                if (
                  rootParam.parentTypes.indexOf(childrenParam.schemaValue) == -1
                ) {
                  childrenParam.children = [];
                  that.deepTreeTableSchemaModel(
                    childrenParam,
                    treeTableModel,
                    rootParam
                  );
                  if (childrenParam.children.length == 0) {
                    childrenParam.children = null;
                  }
                }
              }
            });
          }
        }
      }
    },
    findModelChildren(md, modelData) {
      var that = this;
      if (modelData != null && modelData != undefined && modelData.length > 0) {
        modelData.forEach(function(nmd) {
          if (nmd.pid == md.id) {
            nmd.children = [];
            that.findModelChildren(nmd, modelData);
            //查找后如果没有,则将children置空
            if (nmd.children.length == 0) {
              nmd.children = null;
            }
            md.children.push(nmd);
          }
        });
      }
    },
    initResponseCodeParams() {
      //遍历响应参数
      var that = this;
      var key = Constants.globalTreeTableModelParams + this.swaggerInstance.id;
      //添加自定义属性
      that.multipCode = this.api.multipartResponseSchema;
      that.multipCodeDatas = [];
      //这里不
      that.multipData = {};
      let rcodes = this.api.responseCodes;
      if (rcodes != null && rcodes != undefined) {
        rcodes.forEach(function(rc) {
          //遍历
          if (rc.schema != undefined && rc.schema != null) {
            var respdata = [];
            if (
              rc.responseParameters != null &&
              rc.responseParameters.length > 0
            ) {
              respdata = respdata.concat(rc.responseParameters);
            }
            if (
              rc.responseTreetableRefParameters != null &&
              rc.responseTreetableRefParameters.length > 0
            ) {
              rc.responseTreetableRefParameters.forEach(function(ref) {
                respdata = respdata.concat(ref.params);
              });
            }
            let nrecodedatas = [];
            //遍历得到新的符合antd的树形结构
            if (respdata != null && respdata.length > 0) {
              respdata.forEach(function(param) {
                if (param.pid == "-1") {
                  param.children = [];
                  //判断该参数是否存在schema参数
                  if (param.schema) {
                    //判断当前缓存是否存在
                    var schemaName = param.schemaValue;
                    if (KUtils.checkUndefined(schemaName)) {
                      // //console("schemaValue--checkUndefined");
                      if (that.$Knife4jModels.exists(key, schemaName)) {
                        ////console("存在-不用查找---" + schemaName);
                        ////console(that.$Knife4jModels.instance);
                        var model = that.$Knife4jModels.getByModelName(
                          key,
                          schemaName
                        );
                        if (KUtils.checkUndefined(model)) {
                          var children = model.params;
                          if (KUtils.arrNotEmpty(children)) {
                            children.forEach(function(chd) {
                              var target = that.copyNewParameter(chd);
                              target.pid = param.id;
                              param.children.push(target);
                            });
                          }
                        }
                      } else {
                        ////console("schemavalue--Not Existis");
                      }
                    }
                  }

                  //that.findModelChildren(md, respdata);
                  //查找后如果没有,则将children置空
                  if (param.children.length == 0) {
                    param.children = null;
                  }
                  nrecodedatas.push(param);
                }
              });
            }
            var nresobj = { ...rc, data: nrecodedatas };
            if (!that.multipCode) {
              that.multipData = nresobj;
            }
            that.multipCodeDatas.push(nresobj);
          }
        });
      }
    },
    showResponseEditFieldDescription() {
      //显示说明
      var that = this;
      if (this.api.multipartResponseSchema) {
        //多个
        //默认只显示第1个
        var resp = this.multipCodeDatas[0];
        var id = "knife4jDocumentShowEditor" + that.api.id + resp.code;
        //console("editorShowID:" + id);
        that.showEditorFieldAnyWay(id);
        /* this.multipCodeDatas.forEach(function(resp) {
          var id = "knife4jDocumentShowEditor" + that.api.id + resp.code;
          //console("editorShowID:" + id);
          that.showEditorFieldAnyWay(id);
        }); */
      } else {
        //单个
        var id = "knife4jDocumentShowEditor" + this.api.id;
        this.showEditorFieldAnyWay(id);
      }
    },
    multipartTabCodeChanges(activeKey) {
      var that = this;
      setTimeout(() => {
        var id = "knife4jDocumentShowEditor" + that.api.id + activeKey;
        that.showEditorFieldAnyWay(id);
      }, 1000);
    },
    showEditorFieldAnyWay(containerId) {
      var swaggerInstance = this.swaggerInstance;
      var responseCode = this.api.getHttpSuccessCodeObject();
      var editorContainer = document.getElementById(containerId);
      var paths = [];
      //var aceJsonText = $aceJsonContent.find(".ace_text-layer");
      var aceJsonText = editorContainer.getElementsByClassName(
        "ace_text-layer"
      );
      var acePrintMarginLeft = editorContainer.querySelector(
        ".ace_print-margin"
      ).style.left;
      if (aceJsonText.length > 0) {
        var aceLineDoms = aceJsonText[0].getElementsByClassName("ace_line");
        for (var i = 0; i < aceLineDoms.length; i++) {
          var item = aceLineDoms[i];
          var $variable = item.getElementsByClassName("ace_variable");
          var key;
          if (KUtils.arrNotEmpty($variable)) {
            key = KUtils.toString($variable[0].innerHTML, "").replace(
              /^"(.*)"$/g,
              "$1"
            );
            //判断是否存在
            var sfd = item.getElementsByClassName(
              "knife4j-debug-editor-field-description"
            );
            if (!KUtils.arrNotEmpty(sfd)) {
              var fieldSpan = document.createElement("span");
              fieldSpan.className = "knife4j-debug-editor-field-description";
              fieldSpan.innerHTML = responseCode.responseDescriptionFind(
                paths,
                key,
                swaggerInstance
              );
              fieldSpan.style.left = acePrintMarginLeft;
              item.appendChild(fieldSpan);
            }
          }
          var itemParen = item.getElementsByClassName("ace_paren");
          if (KUtils.arrNotEmpty(itemParen)) {
            //如果元素大于0,拼装多个
            var parentArrs = [];
            for (var e = 0; e < itemParen.length; e++) {
              parentArrs.push(itemParen[e].innerHTML);
            }
            //var parentText = itemParen[0].innerHTML;
            var parentText = parentArrs.join("");
            switch (parentText) {
              case "[":
              case "{":
                paths.push(key ? key : 0);
                break;
              case "}":
              case "]":
                paths.pop();
                break;
            }
          }
        }
      }
    }
  }
};
</script>
<style lang="less" scoped>
.api-tab {
  margin-top: 15px;

  .ant-tag {
    height: 32px;
    line-height: 32px;
  }
}

.api-basic {
  padding: 11px;
}
.api-basic-title {
  font-size: 14px;
  font-weight: 700;
}
.api-basic-body {
  font-size: 14px;
  font-family: -webkit-body;
}

.api-description {
  border-left: 4px solid #ddd;
  line-height: 30px;
}
.api-body-desc {
  padding: 10px;
  min-height: 35px;
  box-sizing: border-box;
  border: 1px solid #e8e8e8;
}
.ant-card-body {
  padding: 5px;
}
.api-title {
  margin-top: 10px;
  margin-bottom: 5px;
  font-size: 16px;
  font-weight: 600;
  height: 30px;
  line-height: 30px;
  border-left: 4px solid #00ab6d;
  text-indent: 8px;
}
.content-line {
  height: 25px;
  line-height: 25px;
}
.content-line-count {
  height: 35px;
  line-height: 35px;
}
.divider {
  margin: 4px 0;
}
</style>
