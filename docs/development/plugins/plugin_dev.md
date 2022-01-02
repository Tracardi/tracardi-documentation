# Plugin development

Creating a plugin in Tracardi as simple as preparing a class that inherits from the 
ActionRunner class and fills several methods. Additionally, to register a plug-in in the system, 
we must prepare a registration method with a set of information about the plug-in, such as 
a name, the plug-in location, etc.

## Dependencies

Plugin can be prepared in external source code and be imported into the system or a file inside Tracardi base code. 
We will use this approach in out tutorial. Regardless of the way we add our plugin we will need tracardi package to use 
it as plugin sdk. Tracardi will install tracardi_plugin_sdk module that we must use to build the plugin.

Install tracardi package by:

```bash
pip install tracardi 
```


## The Plugin Class 

Although the plugin class can be placed anywhere withing the system we will you the default plugins folder in Tracardi.
By default all plugins are placed in `/tracardi/process_engine/action`. Inside this folder you will find folder `v1`. 
`Verion 1 (v1)` is the currently developed version so we will place our plugin in this folder. Let's name it hello_world.py
 

Paste this code to hello_world.py.

```python

from tracardi_plugin_sdk.domain.register import Plugin, Spec, MetaData, Documentation, PortDoc
from tracardi_plugin_sdk.action_runner import ActionRunner
from tracardi_plugin_sdk.domain.result import Result


class HelloWorldAction(ActionRunner):  #(1)

    def __init__(self, *args, **kwargs):  #(2)
        pass

    async def run(self, payload):  #(3)
        payload['hello'] = "Hello world"
        return Result(port="port1", value=payload) #(4)


def register() -> Plugin: #(4)
    return Plugin(
        start=False,
        spec=Spec( # (5)
            module=__name__,
            className='HelloWorldAction',
            inputs=["payload"],  #(6)
            outputs=["port1"], #(7)
            version='0.1',
            license="MIT",
            author="Author"

        ),
        metadata=MetaData( #(8)
            name='Hello world',
            desc='This is hello world',
            icon='plugin',
            group=["Test"],
            documentation=Documentation(
                inputs={
                    "payload": PortDoc(desc="This port takes payload object.")
                },
                outputs={
                    "port1": PortDoc(desc="Appends hello to payload object.")
                }
            )
        )
    )


```

1. This is the plugin class name. It must extend the class ActionRunner.
2. Constructor method is executed when the workflow is initiated. It can not be asynchronous.
3. This method is triggered when the workflow enters the plugin node in the workflow graph. 
4. This is the registration function. 
5. Spec defines the specification of this class, such as module it its placed, class name etc.
6. Spec must define input port's name. There can only be one input port.
7. There can be more than one output port.  
8. MetaData defines plugin none executable data like, node name, icon, group, port documentation, etc.

This class reads input data (payload) and appends "Hello world" to it and returns it on the output port called
"port1". When we restart the system we can add the plugin in Workflow by clicking `+` icon in the left upper corner, and typing 
our plugin module name. It is the path to this plugin `tracardi.process_engine.action.v1.hello_world`. 

Although it is an easy way to load the plugin we want it to auto load and install when the system is restarted.

## Plugin auto loading

In the tracardi api source code go to file `app/setup/on_start.py` and find:


```python

async def add_plugins():
    plugins = [
        'tracardi.process_engine.action.v1.debug_payload_action',
        # ...
    ]
```

and add `tracardi.process_engine.action.v1.hello_world` at the end of the list.

Plugin that once has been registered it will not register unless the version the version of 
the plugin has changes. 

This course of action, while correct, can be very burdensome for a developer. However, this can be remedied. Just set 
the RESET_PLUGIN environment variable to yes when you start Tracardi docker container. 



## Plugin life-cycle

## Scaffold builder

## More advanced use case


