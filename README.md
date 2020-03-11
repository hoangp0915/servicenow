# OnChange table
## Script Include

- Name: GetData
- Client callable: true;
- Script:

```javascript
var GetData = Class.create();
GetData.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {
  getData: function() {
    var queryName = this.getParameter("sysparm_queryName");
    var queryValue = this.getParameter("sysparm_queryValue");
    var object = JSON.parse(this.getParameter("sysparm_object"));
    var table = this.getParameter("sysparm_table");
    var result = {};
    var gr = new GlideRecord(table);
    gr.addQuery(queryName == null ? "sys_id" : queryName, queryValue);
    gr.query();
    if (gr.next()) {
      for (var key in object) {
        if (object.hasOwnProperty(key)) {
          result[key] = gr.getValue(object[key]);
        }
      }
    }
    return JSON.stringify(result);
  },
  type: "GetData"
});
```

## Client script

- Name: [Test] onChange field instance
- Table: VM Information
- Type: onChange
- Field name: Instance (Amazon EC2 Instance)
- Script:

```javascript
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    var object = {
        vcpu: 'vcpu',
        memory: 'memory',
        storage: 'storage',
        network: 'network',
    };

    if (isLoading || newValue === '') {
        setDataForm({}, object);
        return;
    }

    var gAjax = new GlideAjax('GetData');
    gAjax.addParam('sysparm_name', 'getData');
    gAjax.addParam('sysparm_queryValue', newValue);
    gAjax.addParam('sysparm_table', 'x_hmsp_gcs_tester_vm_instances');
    gAjax.addParam('sysparm_object', JSON.stringify(object));
    gAjax.getXMLAnswer(function(responce) {
        var result = JSON.parse(responce);
        console.log('result: ', result);
        setDataForm(result, object);
    });
}

function setDataForm(result, object) {
    for (var key in object) {
        if (object.hasOwnProperty(key)) {
            var element = object[key];
            var check = Object.keys(result).length === 0 ? '' : result[element];
            g_form.setValue(element, check, check);
        }
    }
}
});
```
# Hide
## Script Include:

- Name: CheckExist
- Client callable: true
- Code:

```javascript
var CheckExist = Class.create();
CheckExist.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {
  checkExist: function(sysId, table) {
    var gr = new GlideRecord(table);
    gr.addQuery("change_request", sysId);
    gr.query();
    if (gr.hasNext()) {
      return true;
    }
    return false;
  },
  type: "CheckExist"
});
```

## UI Action

- Name: New
- Table: VM Information
- Action name: sysverb_new
- List banner button: true;
- Condition: Design: parent.state = 5

```javascript
	!(new CheckExist().checkExist(parent.sys_id, "x_hmsp_gcs_tester_vm_information")) && parent.state != -5
```

- Script:

```javascript
var uri = action.getGlideURI();
var path = "x_hmsp_gcs_tester_vm_information.do";
uri.set("sys_id", "-1");
action.setRedirectURL(uri.toString(path));
action.setNoPop(true);
```


# Other
sys_schema_attribute :Dictionary Attribute
## Attributes
```
	no_audit=true,staticDependent=u_for_test
```
