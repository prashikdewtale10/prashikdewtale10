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






