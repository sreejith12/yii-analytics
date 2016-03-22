# Using the Google Analytics Extension #

This wiki is about how to use the Google Analytics extension in Yii. You must first [install this extension](GoogleAnalyticsInstallation.md).

Table of Contents:


## Accessing Google Analytics in Yii ##
Since the Google Analytics extension is setup as a component, you can simply use the following call to access the extension:
```
Yii::app()->googleAnalytics
```


## Calling a Google Analytics Method ##
In Google Analytics, you call various [methods](https://developers.google.com/analytics/devguides/collection/gajs/methods/) to change the settings and values that are passed to Google's severs. For the Yii extension, you use a similar setup. All you need to do is call the name of the method, and pass in the parameters (not as an array!)

### A simple example ###
A normal call to set a custom variable in JavaScript:
```
_gaq.push(['_setCustomVar', 1, 'Section', 'Life & Style', 3]);
```

Within a controller or view, you can do the same as above via the extension:
```
Yii::app()->googleAnalytics->_setCustomVar(1, 'Section', 'Life & Style', 3);
```

### A more complex example ###
Sometimes you need to push quite a bit of data into Google Analytics. With this extension, that is fairly easy.

For an example, let's push in a transaction when the user completes a checkout via the `checkout` action within the `cart` controller. You can see within this example that Yii's relational records can be used (see: `$order->Store->Name`)

_`protected/controllers/cart.php`_:
```
protected function beforeRender($view)
{
    $return = parent::beforeRender($view);
    Yii::app()->googleAnalytics->render();
    return $return;
}

public function actionCheckout( )
{
    // Do some processing here (let's say $order has information about the order)
    if($order->isComplete)
    {
        // Start the transaction using $order's information
        Yii::app()->googleAnalytics->_addTrans(
                                        $order->OrderID, 
                                        $order->Store->Name, 
                                        $order->Total, 
                                        $order->Tax, 
                                        $order->ShippingAmount, 
                                        $order->City, 
                                        $order->State, 
                                        $order->Country
                                    );
        
        // Loop through each item that the order had
        foreach($order->Items as $item)
        {
            // And add in the item to pass to Google Analytics
            Yii::app()->googleAnalytics->_addItem(
                                            $order->OrderID,
                                            $item->SKU,
                                            $item->Name,
                                            $item->Category->Name,
                                            $item->Price,
                                            $item->Quantity
                                        );
        }
        
        // Finally, call _trackTrans to finalize the order.
        Yii::app()->googleAnalytics->_trackTrans();
    }
}

```

### Disallowed methods ###
Nearly all of the [methods](https://developers.google.com/analytics/devguides/collection/gajs/methods/) are accessible via the extension. The exceptions are as follows:
  * Any deprecated method
  * Any method starting with `_get`
  * `_link` (see: [issue #2](https://code.google.com/p/yii-analytics/issues/detail?id=#2))
  * `_linkByPost` (see: [issue #3](https://code.google.com/p/yii-analytics/issues/detail?id=#3))


## Rendering the Google Analytics Output ##
Rendering within the extension depends on the way you [configured it](GoogleAnalyticsInstallation#Step_3:_(Optional)_Add_in_auto-render.md).

### If Auto Rendering  is enabled ###
If auto rendering is enabled and you followed the configuration steps (adding `beforeRender` call to your controllers) then there is nothing else for you to do to render the JavaScript code.

### If Auto Rendering  is disabled ###
If you have auto rendering disabled (which it is by default), then you can call the `render()` method within your views which will return the rendered Google Analytics JavaScript code. In almost all cases, you should use this in your main layout views (e.g. `protected/views/layouts/main.php`)
```
<script type="text/JavaScript">
<?php echo Yii::app()->googleAnalytics()->render(); ?>
</script>
```

**Note**: The `render` method does not wrap `<script></script>` tags around the output. If auto-rendering is enabled, [CClientScript::registerScript](http://www.yiiframework.com/doc/api/1.1/CClientScript#registerScript-detail) is utilized, otherwise JavaScript code is returned.