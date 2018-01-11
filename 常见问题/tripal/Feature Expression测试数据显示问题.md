# Feature Expression测试数据显示问题

> 安装`tripal_analysis_expression`后修改测试数据显示

涉及到两个文件

`sites/all/modules/tripal_analysis_expression/includes/feature_heatmap_form.inc`
`sites/all/modules/tripal_analysis_expression/js/example.js`

* 找到`feature_heatmap_form.inc`, 修改`description`及`placeholder`

```php
$form['heatmap_feature_uniquename'] = array(
    '#type' => 'textarea',
    '#title' => t('Enter feature unique names'),
    '#description' => 'Example feature unique names: mRNA1235,mRNA26658,mRNA889,mRNA11337,mRNA24362,mRNA14232,mRNA9079,mRNA7637,mRNA17140,mRNA8967,mRNA25955,mRNA9018,mRNA14656,mRNA408',
    '#attributes' => array(
      'placeholder' => 'Example:mRNA1235,mRNA26658,mRNA889,mRNA11337,mRNA24362,mRNA14232,mRNA9079,mRNA7637,mRNA17140,mRNA8967,mRNA25955,mRNA9018,mRNA14656,mRNA408'
    ),
  );
```

* 找到`example.js`,修改`example`值

```js
Drupal.behaviors.tripal_analysis_expression_example = {
    attach: function (context, settings) {
      var example = 'mRNA1235,mRNA26658,mRNA889,mRNA11337,mRNA24362,mRNA14232,mRNA9079,mRNA7637,mRNA17140,mRNA8967,mRNA25955,mRNA9018,mRNA14656,mRNA408';
      $('#edit-example-button').click(function(e) {
        e.preventDefault()
        $(this).val('Creating example heat map. Please wait...')
        $('#edit-heatmap-feature-uniquename').val(example)
        $(this).parents('form').submit()
      })
    }
  }
```

