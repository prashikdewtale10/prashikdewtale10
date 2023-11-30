### Hi there 👋 this is Prashik Dewtale
import json
import os
import tempfile
import flock

def update_json_file(file_path, update_function):
    lock_file_path = file_path + ".lock"

    with flock.Flock(lock_file_path, timeout=10):
        # Step 2: Load JSON file into native Python data structure
        with open(file_path, 'r') as json_file:
            data = json.load(json_file)

        # Step 3: Perform manipulations on the data structure
        updated_data = update_function(data)

        # Step 4: Convert the updated data structure to JSON
        updated_json = json.dumps(updated_data, indent=2)

        # Step 5: Create a new JSON file
        temp_file_path = tempfile.mktemp(suffix=".json")
        with open(temp_file_path, 'w') as temp_file:
            temp_file.write(updated_json)

        # Step 6: Move the new file over the old version
        os.replace(temp_file_path, file_path)

    # Step 7: Release the lock (automatically done when leaving the 'with' block)

=----------------------------------------------
def update_plugins_configurations(plugin_name, logger):

    '''
        **Type** Public

        **Arguments:** 
            - plugin_name: name of plugin with its context to be updated
            - logger: logger obeject to add log messge to logger file

        **Returns:** None
            
        Function is use to update the configurations of specified plugin name with its context.
    '''
    # To get function name
    frame = inspect.currentframe()
    logger.info("-"*100)
    logger.info(f"Inside {frame.f_code.co_name}")
    # getting plugin name and updation context by splitting by space.
    plugin_context = plugin_name[1]
    plugin_name = plugin_name[0]
    context_list = plugin_context.split(",")

    # Permission for updating file 
    Update_file = False
    # Aquring lock

    try:
        if (plugin_context != None and plugin_context !="") and (plugin_name != None):
            # Acquring lock
            logger.info("aquiring lock")
            lock.acquire()
            logger.info("aquired lock")
            # Reading json
            with open(JSON_FILE_PATH, 'r') as f:
                json_data = json.load(f)
                plugins = json_data.get("PLUGINS_CONF")
                # configurarion exists
                exits = False
                for plugin in plugins:
                    if plugin.split(".")[2] == plugin_name:
                        # checking if resouce or parameter exists or not for updation
                        if plugins.get(plugin) != []:
                            for plugin_item in plugins.get(plugin):
                                if plugin_item.get('PARAM_CODE') and plugin_item.get('RESOURCE_CODE'):
                                    # looping context list for matching param code 
                                    for context in context_list:
                                        if context.split("=")[0] == "PARAM_CODE" and context.split("=")[1].replace("'", "") == plugin_item.get('PARAM_CODE'): 
                                            for context_info in context_list:  
                                            # Checking for matching resource code 
                                                if context_info.split("=")[0] == "RESOURCE_CODE" and context_info.split("=")[1].replace("'", "") == plugin_item.get('RESOURCE_CODE'):
                                                    exits = True
                                                    # Updating configurations for existing 
                                                    Update_file = True
                                                    data = plugins.get(plugin)
                                                    logger.info(f"Updating configuration for Param code {plugin_item.get('PARAM_CODE')} and Resource code {plugin_item.get('RESOURCE_CODE')}")
                                                    for item in data:
                                                        if f"{item.get(context_info.split('=')[0])}" == context_info.split("=")[1].replace("'", ""):
                                                            for con in context_list:
                                                                if con.split('=')[0] not in ['PARAM_CODE', 'RESOURCE_CODE', 'CRON_PATTERN']:
                                                                    item['CONTEXT'][con.split('=')[0]] = con.split('=')[1].replace("'", "")
                                                                else:
                                                                    if item.get(con.split('=')[0]) !=None:
                                                                        item[con.split('=')[0]] = con.split('=')[1].replace("'", "")

                        # For adding configuartions which does not exists
                        try:
                            # Checking for mandetory keys
                            keys_exists = False
                            cron_exists = False
                            fqn_parts = plugin.split('.')
                            module = '.'.join(fqn_parts[:-1])
                            module_name = importlib.import_module(module)
                            mandetory_keys = module_name.MANDETORY_CONTEXT_KEYS
                            if not(exits):
                                if mandetory_keys != []:
                                    for key in mandetory_keys:
                                        for context in context_list:
                                            if context.split("=")[0] == mandetory_keys[0]:
                                                keys_exists = True
                                            if context.split("=")[0] == "CRON_PATTERN":
                                                cron_exists = True
                                else:
                                    # for plugin with mandetory_keys == []
                                    keys_exists = True

                                if keys_exists:
                                    Update_file = True
                                    # logger.info(f"Adding configuration for Param code {plugin_item.get('PARAM_CODE')} and Resource code {plugin_item.get('RESOURCE_CODE')}")
                                    data = plugins.get(plugin)
                                    add_config = dict()

                                    for context in context_list:
                                        # *check for mandetory keys 
                                        if context.split('=')[0] not in ['PARAM_CODE', 'RESOURCE_CODE', 'CRON_PATTERN']:
                                            if mandetory_keys != []:
                                                if context.split('=')[0] in mandetory_keys:
                                                    add_config['CONTEXT'] = {
                                                        context.split('=')[0] : context.split('=')[1].replace("'", "")
                                                    }
                                            else:
                                                # adding empty context if no mandetory key exists for plugin
                                                add_config['CONTEXT'] = {
                                                        
                                                    }
                                        else:
                                            add_config[context.split('=')[0]] = context.split('=')[1].replace("'", "")

                                    # # adding empty context if no mandetory key exists for plugin
                                    if mandetory_keys == []:
                                        add_config['CONTEXT'] = { }
                                    if not(cron_exists):
                                        add_config['CRON_PATTERN'] = module_name.DEFAULT_CRON_PATTERN
                                    data.append(add_config)

                                else:
                                    raise AttributeError

                            else:
                               # Message for already existed configuarations
                               logger.error(f"Configurations alerady exists for plugin {plugin_name} with context {plugin_context}")
                               print(f"Configurations alerady exists for plugin {plugin_name} with context {plugin_context}")  

                        except AttributeError as e:
                            logger.warning(f"AttributeError occured due {mandetory_keys[0][0]} is not present in context which is required")
                            print(f"{mandetory_keys[0][0]} is required while adding configuration for {plugin_name} in context")
                                                 
            if Update_file and not(exits):
                temp_file_path = tempfile.mktemp(suffix=".json")
                logger.info("Writing updated data in temp file")
                with open(temp_file_path, 'w') as temp_file:
                    json.dump(json_data, temp_file, indent=4)
                logger.info("replacing conf.json file context with temp file context")
                # Writing json with updated context
                os.replace(temp_file_path, JSON_FILE_PATH)
                logger.info(f"Configurations updated for plugin {plugin_name} with context {plugin_context}")
                print(f"Configurations updated for plugin {plugin_name} with context {plugin_context}")
                
            # Releasing lock
            logger.info("releasing aquired lock")
            lock.release()
            logger.info("released aquired lock")

        else:
            raise ValueError

    except ValueError as e:
        if plugin_context == None or plugin_context == "":
            print("Plugin context cannot be empty, provide plugin context along with plugin name")
            logger.error(f"ValueError occur while updating configurations for plugin {plugin_name} due to invalid plugin context")
        else:
            print("Provide valid plugin name and its context")
            logger.error(f"Error occur while updating configurations for plugin {plugin_name} with context {plugin_context}", e)
    except Exception as e:
        logger.error(f"Exception occur while updating configurations for plugin {plugin_name} with context {plugin_context}", e)





