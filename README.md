import json
import inspect
import importlib
import tempfile
import os

def update_plugins_configurations(plugin_name, logger, lock):
    frame = inspect.currentframe()
    logger.info("-" * 100)
    logger.info(f"Inside {frame.f_code.co_name}")

    plugin_context = plugin_name[1]
    plugin_name = plugin_name[0]
    context_list = plugin_context.split(",")

    Update_file = False

    try:
        if plugin_context and plugin_name:
            logger.info("acquiring lock")
            lock.acquire()
            logger.info("acquired lock")

            with open(JSON_FILE_PATH, 'r') as f:
                json_data = json.load(f)
                plugins = json_data.get("PLUGINS_CONF")
                exits = False

                for plugin in plugins:
                    if plugin.split(".")[2] == plugin_name:
                        if plugins.get(plugin):
                            for plugin_item in plugins[plugin]:
                                if all(
                                    plugin_item.get('PARAM_CODE'),
                                    plugin_item.get('RESOURCE_CODE')
                                ):
                                    for context in context_list:
                                        param_match = (
                                            context.split("=")[0] == "PARAM_CODE"
                                            and context.split("=")[1].replace("'", "") == plugin_item.get('PARAM_CODE')
                                        )
                                        resource_match = (
                                            context.split("=")[0] == "RESOURCE_CODE"
                                            and context.split("=")[1].replace("'", "") == plugin_item.get('RESOURCE_CODE')
                                        )
                                        if param_match and resource_match:
                                            exits = True
                                            Update_file = True
                                            data = plugins[plugin]
                                            logger.info(f"Updating configuration for Param code {plugin_item.get('PARAM_CODE')} and Resource code {plugin_item.get('RESOURCE_CODE')}")
                                            for item in data:
                                                if item.get(context.split('=')[0]) == context.split("=")[1].replace("'", ""):
                                                    for con in context_list:
                                                        if con.split('=')[0] not in ['PARAM_CODE', 'RESOURCE_CODE', 'CRON_PATTERN']:
                                                            item['CONTEXT'][con.split('=')[0]] = con.split('=')[1].replace("'", "")
                                                        elif item.get(con.split('=')[0]) is not None:
                                                            item[con.split('=')[0]] = con.split('=')[1].replace("'", "")

                try:
                    keys_exists = False
                    cron_exists = False

                    if not exits:
                        if module_name.MANDETORY_CONTEXT_KEYS:
                            for key in module_name.MANDETORY_CONTEXT_KEYS:
                                if any(context.split("=")[0] == key for context in context_list):
                                    keys_exists = True
                                if any(context.split("=")[0] == "CRON_PATTERN" for context in context_list):
                                    cron_exists = True
                        else:
                            keys_exists = True

                        if keys_exists:
                            Update_file = True
                            data = plugins.get(plugin)
                            add_config = {'CONTEXT': {}}

                            for context in context_list:
                                if context.split('=')[0] not in ['PARAM_CODE', 'RESOURCE_CODE', 'CRON_PATTERN']:
                                    if module_name.MANDETORY_CONTEXT_KEYS and context.split('=')[0] in module_name.MANDETORY_CONTEXT_KEYS:
                                        add_config['CONTEXT'][context.split('=')[0]] = context.split('=')[1].replace("'", "")
                                else:
                                    add_config[context.split('=')[0]] = context.split('=')[1].replace("'", "")

                            if not cron_exists:
                                add_config['CRON_PATTERN'] = module_name.DEFAULT_CRON_PATTERN
                            data.append(add_config)

                        else:
                            raise AttributeError

                    else:
                        logger.error(f"Configurations already exist for plugin {plugin_name} with context {plugin_context}")
                        print(f"Configurations already exist for plugin {plugin_name} with context {plugin_context}")

                except AttributeError as e:
                    logger.warning(f"AttributeError occurred: {module_name.MANDETORY_CONTEXT_KEYS[0][0]} is not present in context, which is required")
                    print(f"{module_name.MANDETORY_CONTEXT_KEYS[0][0]} is required while adding configuration for {plugin_name} in context")

            if Update_file and not exits:
                temp_file_path = tempfile.mktemp(suffix=".json")
                logger.info("Writing updated data in temp file")
                with open(temp_file_path, 'w') as temp_file:
                    json.dump(json_data, temp_file, indent=4)
                logger.info("Replacing conf.json file context with temp file context")
                os.replace(temp_file_path, JSON_FILE_PATH)
                logger.info(f"Configurations updated for plugin {plugin_name} with context {plugin_context}")
                print(f"Configurations updated for plugin {plugin_name} with context {plugin_context}")

            logger.info("Releasing acquired lock")
            lock.release()
            logger.info("Released acquired lock")

        else:
            raise ValueError

    except ValueError as e:
        if not plugin_context:
            print("Plugin context cannot be empty, provide plugin context along with plugin name")
            logger.error(f"ValueError occurred while updating configurations for plugin {plugin_name} due to invalid plugin context")
        else:
            print("Provide valid plugin name and its context")
            logger.error(f"Error occurred while updating configurations for plugin {plugin_name} with context {plugin_context}", e)
    except Exception as e:
        logger.error(f"Exception occurred while updating configurations for plugin {plugin_name} with context {plugin_context}", e)
