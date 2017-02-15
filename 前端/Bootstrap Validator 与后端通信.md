##Bootstrap Validator
###通过 AJAX 与后台服务器交互

###bootstrapValidator 接收后台服务器返回的验证消息
* 前端只能对输入数据的格式等进行验证，但是例如：用户名是否存在，密码是否正确等信息的验证需要由服务器来执行
* 由于使用了 AJAX 与后台交互，因而后台对数据进行验证后，返回的也是 JSON 数据，故需要 Bootstrap Validator 对返回的消息进行分析并更新表单的显示

以下内容截取自 [Showing custom message returned from server](http://http://formvalidation.io/examples/showing-custom-message-returned-from-server/#step-1-defining-the-validation-rules)

#####使用的方法
######updateMessage：
updateMessage(field*, validator*, message*): FormValidation — Update the error message.

|Parameter|Type|Description|
|-|-|
|field|String/jQuery|The field name or field element|
|validator|String|The validator name|
|message|String|The error message|

######updateStatus：
updateStatus(field*, status*, validator): FormValidation — Update validator result for given field

|Parameter|Type|Description|
|-|-|
|field|String/jQuery|The field name or field element|
|status|String|Can be `NOT_VALIDATED`, `VALIDATING`, `INVALID` or `VALID`|
|validator|String|The validator name. If `null`, the method updates validity result for all validators|

#####步骤：
######Step 1: Defining the validation rules
In addition to usual validators, we also attach a special validator called blank to each field which need to show the custom message returned from the server.

The blank validator doesn't have any option:

	$('#signupForm').formValidation({
        fields: {
            username: {
                validators: {
                    notEmpty: {
                        ...
                    },
                    stringLength: {
                        ...
                    },
                    regexp: {
                        ...
                    },
                    // The bank validator doesn't have any option
                    blank: {}
                }
            },
            email: {
                validators: {
                    notEmpty: {
                        ...
                    },
                    emailAddress: {
                        ...
                    },
                    blank: {}
                }
            },
            password: {
                validators: {
                    notEmpty: {
                        ...
                    },
                    blank: {}
                }
            }
        }
    });

Since the blank validator always returns true, the field is supposed to pass it whenever the validation is performed in the client side.

We will see how we update the validation result later.

######Step 2: Submitting the form data via Ajax
When all fields satisfy the validation rules, we can trigger the success.form.fv event to send the form data to server via an Ajax request:

    $('#signupForm')
        .formValidation({
            ...
        })
        .on('success.form.fv', function(e) {
            // Prevent default form submission
            e.preventDefault();

            var $form = $(e.target),                    // The form instance
                fv    = $form.data('formValidation');   // FormValidation instance

            // Send data to back-end
            $.ajax({
                url: '/path/to/your/back-end/',
                data: $form.serialize(),
                dataType: 'json'
            }).success(function(response) {
                // We will display the messages from server if they're available
            });
        });

The error messages returned from server will be processed inside the success handler of the Ajax request. We will see how to do that in the next step.

######Step 3: Showing message returned from the server
After getting the data sent from the client via the Ajax request, the server will perform validation using certain programming language. Depend on the validation result, it might response an encoded JSON as

    // A sample response if all fields are valid
    {
        "result": "ok"
    }

or

    // A sample response if there's an invalid field.
    // It also tell which and the reason why the field is not valid
    {
        "result": "error",
        "fields": {
            "username": "The username is not available",
            "password": "You need to have more secure password"
            ...
        }
    }


Lastly, we can use the updateMessage() and updateStatus() methods to set the message and validation result of the blank validator:

    $.ajax({
        url: '',
        data: $form.serialize(),
        dataType: 'json'
    }).success(function(response) {
        // If there is error returned from server
        if (response.result === 'error') {
            for (var field in response.fields) {
                fv
                    // Show the custom message
                    .updateMessage(field, 'blank', response.fields[field])
                    // Set the field as invalid
                    .updateStatus(field, 'INVALID', 'blank');
            }
        } else {
            // Do whatever you want here
            // such as showing a modal ...
        }
    });