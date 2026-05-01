# 02-JavaScript常见使用方法

# JavaScript常见使用方法

##### 表单页面常见状态

```javascript

 var activityCode = that.workflowInfo.activityCode; 
 console.log("当前表单流程节点",activityCode,"当前表单是否已经被提交过", this.submited, "当前表单是否为新建表单", this.isNew,
            "当前表单是否为草稿状态", this.isDraft, "当前表单是否进入了编辑状态", this.inEdit, "当前表单的打开方式是否为移动端打开", this.isMobile,
            "当前表单是否可提交/同意/保存", this.formInEdit, "当前用户信息", this.currentUser)
```

##### 流程页面新增、草稿、流程中等判断条件

```java

新增、草稿 编辑情况下
 if (!this.submited || this.isNew || this.isDraft) {}
流程不在发起节点或者页面不在编辑状态
    if (activityCode !== "Activity2" || !this.inEdit || !this.formInEdit) {
        } 
```

##### CSS样式

```css

print 打印
forward 转办
copy 复制
cancel 作废流程

.ant-btn.ant-btn-default.print {
    display: none !important;
}

.ant-btn.ant-btn-default.forward {
    display: none !important;
}

.ant-btn.ant-btn-default.cancel {
    display: none !important;
}

.ant-btn.ant-btn-default.copy {
    display: none !important;
}

```

##### 系统调用方法

```javascript

//系统使用的保存数据和更新，不给id是保存一条新数据，给id是更新对应id数据
axios.post("/api/runtime/form/save", {
    "bizObject": {
        "data": {
            "id": taskid,
            "ACTUALSTART_": newTime,
            "taskState": "进行中",
            "NOTES_": this.remark.value,
            "ACTUALFINISH_": newTime2,

        },
        "schemaCode": 'M0Task', // 业务模型编码
        "sheetCode": "M0Task" // 业务模型编码
    }
}).then(res => {})

//系统使用的提交数据
axios.post("/api/runtime/form/submit", {
  bizObject: {
    data: params,
    schemaCode: code,
    sheetCode: code,
    id: "",
  },
  formType: "2"
}).then((res) => {
});
```

‍

‍

##### 基本调用方法

```javascript

var that;
var activityCode;
var activeUserId;
that = this;

activeUserId = data.creater[0].id
console.log("渲染完成后", data, activeUserId);

const workflowInfo = that.workflowInfo;
if (workflowInfo) {
    activityCode = workflowInfo.activityCode;
}

var currentUser=this.currentUser;//当前用户

//基本调用业务集成

axios.post("/api/bizservice/method_ext/invoke_biz_service_method", {
    "code": "getM0projectEngineeringData",//方法编码
    "serviceCode": "PeriodicPlanApproval",//服务编码
    "testInputParametersMap": {//参数
        "joinProjectEngineering": e.data.params.joinProjectEngineering.id
    }
}).then(function (res) {
    console.log("项目表", res)
})

//调用业务规则

axios.post("/api/v2/businessRule/invoke", {
    "appCode": "scheduleManagement",//应用编码
    "schemaCode": "weeklyPlanApproval",//表单编码
    "businessRuleCode": "getPlanList",//业务规则
    "data": { "id": data.id }//参数
}).then(function (res) {
    console.log("项目表", res)
})

```

##### 前端js获取当前用户的用户信息

```bash

        const response = await axios.post('/api/v2/businessRule/invoke', {
            "appCode": "CommonUtils",
            "schemaCode": "SystemCommonUtils",
            "businessRuleCode": "getPersonnelContent",
            "data": {
                "personId": that.currentUser.id,
                "param": "id,joinEngineeringContract,joinProjectEngineering,joinCompanyUnit,joinUnitType,role,joinCompanyUnitProjDep"
            }
        });
```

##### 通过pid和id建立一个符合树形结构的数据

```java

  const treeToData = (list, id) => {
        const res = [];
        list.forEach((item) => {
            if (item.pid === id) {
                let copy_arr = treeToData(list, item.id)
                item.children = copy_arr.length > 0 ? copy_arr : null;
                res.push(item);
            }
        });
        res.sort((a, b) => a['branchSubitemCode'] - b['branchSubitemCode']);
        return res;
    }
```

##### 将树形转成扁平数组

```java

  function flattenTree(tree) {
        let flatArray = [];

        // 递归遍历树形结构
        function flattenNode(node) {
            // 复制节点并删除 children 字段
            const { children, ...flattenedNode } = node; // 使用解构赋值删除 children
            flatArray.push(flattenedNode); // 添加扁平化后的节点

            // 如果节点有 children，递归处理子节点
            if (children && children.length > 0) {
                children.forEach(flattenNode); // 遍历子节点
            }
        }

        // 遍历根节点数组
        tree.forEach(flattenNode);
        return flatArray;
    }
```

##### 催办和列表js脚本

```javascript

//get请求
this.axios.get('/url').then(res => {
    alert(res.errmsg)
});
//post请求
let params = {
    instanceId: "0bf05ff89d4e4c82b28bd460a9da7f83",
    text: "",
    urgeType: 1,
}
this.axios.post('/api/runtime/urge/saveDing', params).then(res => {
    console.log("返回res", res)
    alert(res.errmsg)
});

axios
    .post('/api/runtime/urge/saveDing', {
        instanceId: record.flowID,
        text: record.taskName,
        urgeType: 1,
    })

console.log("scriptData", scriptData)

let that = this
let params1 = {
    "code": "getDisciplineCommitteeRecData",
    "serviceCode": "CleanResourceLearningPus",
    "testInputParametersMap": {}
}
that.axios.post('/api/bizservice/method_ext/invoke_biz_service_method', params1).then(res => {
    console.log("返回履职工作记录的流程id", res)
    let data = res.data.data.data
    console.log("data", data)
    if (data.length > 0) {
        for (let i = 0; i < data.length; i++) {
            let params = {
                instanceId: data[i].workflowInstanceId,
                text: "请填写履职工作记录",
                urgeType: 1,
            }
            that.axios.post('/api/runtime/urge/saveDing', params).then(res => {
                console.log("返回流程催办成功信息", data[i].workflowInstanceId)
            });
        }
        // alert("催办成功")
        that.$message.success('催办成功')
    }
});

console.log("当行数据", scriptData)
let data = scriptData[0]
console.log("当行数据", data)

params = {
    instanceId: data.workflowInstanceId,
    text: "请填写履职工作记录",
    urgeType: 1,
}
this.axios.post('/api/runtime/urge/saveDing', params).then(res => {
    console.log("返回流程催办成功信息", data.workflowInstanceId)
});
this.$message.success('催办成功')
```

##### 加载遮罩

```javascript

    /**
* 显示加载弹窗(硬编码)
* @param (String) selector 弹窗外部容器选择器
* @param (String) id 弹窗ID
* @return void 返回空
*/
function showLoadingHardCode(selector, id, text) {
    const shell = document.querySelector(selector);
    const dialog = document.createElement("div");
    dialog.classList.add("next-overlay-wrapper");
    dialog.classList.add("opened");
    dialog.style.position = "fixed";
    dialog.style.left = 0;
    dialog.style.top = 0;
    dialog.style.bottom = 0;
    dialog.style.right = 0;
    dialog.style["z-index"] = 10002;
    dialog.style.display = "flex";
    dialog.style["justify-content"] = "center";
    dialog.style["align-items"] = "center";
    dialog.style.background = "rgba(0,0,0,.45)";
    dialog.id = id;
    dialog.innerHTML = `
        <div>
            </div>

                <div class="vc-div div_lezc5ky0" style="display: flex;align-items: center;justify-content: center;color:white">
                    <i class="next-icon next-icon-loading next-medium vc-icon icon_lezc5ky1"></i>

                    <div class="vc-text text_lezc5ky2" title="">${text}...</div>

                </div>

                </div>

            </div>

        </div>`;

    shell.appendChild(dialog);
}
/**
 * 隐藏加载弹窗(硬编码)
 * @param (String) selector 弹窗外部容器选择器
 * @param (String) id 弹窗ID
 * @return void 返回空
 */
function hideLoadingHardCode(selector, id) {
    const shell = document.querySelector(`${selector} #${id}`);
    if (shell) shell.parentNode.removeChild(shell);
}

showLoadingHardCode("#app", "did", "请稍等...");
hideLoadingHardCode("#app", "did");

//根据控件隐藏后将字段置空

```

##### 按钮点击事件

```javascript

setBtn(this, data);
const setBtn = (that, data) => {
    let xxxx = document.querySelector(".xxxx");
    xxxx.addEventListener("click", () => {
        console.log('click')

    })
}
```

##### 控件值变化事件

```javascript

this.txtName.valueChange.subscribe(function (change) {
    //最新值
    console.log(change.value);
    //旧值
    console.log(change.oldValue);
});

```

##### 获取url参数

```javascript

function getQueryParam(param) {
    let queryString = window.location.search.substring(1);
    let params = queryString.split("&");
    let result = {};
  
    params.forEach(param => {
      let [key, value] = param.split("=");
      result[key] = decodeURIComponent(value);
    });
  
    return result[param];
  }

```

##### 日期处理

```javascript

//将日期格式化yyyy-mm-dd
function formatDate(inputDate) {
    const date = new Date(inputDate); // 将字符串解析为日期对象
  
    // 获取日期的年、月、日
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0'); // 月份需要+1，并补零
    const day = String(date.getDate()).padStart(2, '0'); // 日补零
  
    // 格式化为 yyyy-MM-dd
    const formattedDate = `${year}-${month}-${day}`;
    return formattedDate;
  }
  
  //获取当前第几周
  function getWeekNumber(d) {
    // 将日期设置为当天的00:00:00
    d.setHours(0, 0, 0, 0);
    // 将星期天设为一周的第一天
    d.setDate(d.getDate() + 4 - (d.getDay() || 7));
    // 获取年份的第一天
    var yearStart = new Date(d.getFullYear(), 0, 1);
    // 计算日期到年初的天数，并加上4天（周的偏移量）
    var weekNo = Math.ceil((((d - yearStart) / 86400000) + 1) / 7);
    return weekNo;
  }
```

##### ‍弹窗

```javascript

// 成功
this.$message.success('success');
// 失败
this.$message.error('error');
// loading
this.$message.loading();
this.$message.loading('加载中');
//0不自动隐藏，大于0显示的秒数
var closeLoading = this.$message.loading('加载中', 0);
closeLoading();

this.$confirm({ 
    title:'对话框',
    content:'ddddd',
    onOk(){ 
        alert('onOk') 
    },
    onCancel(){
        alert('onCancel')
    }
})
```
