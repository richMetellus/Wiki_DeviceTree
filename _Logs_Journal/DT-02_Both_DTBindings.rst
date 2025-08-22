#################################################
Common Zephyr and Linux Device Tree Bindings
#################################################



****************
``gpio-keys``
****************


In some device tree, you might see something like this

.. code-block:: c

    / {
    
    	gpio_keys {
    		compatible = "gpio-keys";
    		user_button: button_0 {
    			label = "User";
    			gpios = <&gpioc 13 GPIO_ACTIVE_HIGH>;
    		};
    	};
    
    }
    
    

* The node ``gpio-keys`` serves as a container to contains all the GPIO devices as a subnode(user_button) that
  are input. 
* The ``compatible = "gpio-keys"`` is a generic driver

    * in **zephyr**: the location is: ``zephyr/drivers/input/input_gpio_keys.c``
        
        * the generic driver is written by folks from Google
        * there is a function ``static int gpio_keys_init(const struct device *dev)``
          that get called for each sub-nodes in the ``gpio-keys`` bucket as indicated by that
          macro ``DT_INST_FOREACH_STATUS_OKAY(GPIO_KEYS_INIT)``
        
            * this static function will configure the flag as GPIO_INPUT in ``ret = gpio_pin_configure_dt(gpio, GPIO_INPUT);``
                
                * This ends up calling the sub-node specific device driver (driver associate with &gpioc) 
                  and configure the pin accordingly.