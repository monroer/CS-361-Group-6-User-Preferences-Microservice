# CS-361-Group-6-User-Preferences-Microservice


Overview
This microservice saves and retrieves per-user preferences for any application. Preferences are stored as simple key-value pairs in a shared JSON file, organized by username. There are no hardcoded preference keys, your application defines its own defaults and validation rules, making this microservice reusable across any project.
It provides four public functions:

get_preferences() — load a user's saved preferences, merged with your app's defaults
update_preference() — save or update a single preference value
reset_preferences() — wipe all of a user's saved preferences back to defaults
delete_preference() — remove one specific preference key for a user


Storage Format
All preferences are saved in a single JSON file (user_preferences.json by default). The file is structured as a dictionary of usernames, each mapping to their saved key-value pairs.
{
  "alice": {
    "theme": "light",
    "items_per_page": 50
  },
  "bob": {
    "theme": "dark"
  }
}
Only overridden values are stored. Preferences the user has never changed are not written to the file, they are filled in from your app's defaults at read time.


Functions
get_preferences(username, defaults=None, prefs_file=PREFS_FILE)
Loads a user's saved preferences and merges them with your application's defaults. Any key present in defaults but not yet saved by the user will appear in the result with its default value.
Parameters:
Parameter  Type  Required  Description
username   str   Yes       The user whose preferences to load
defaults   dict  No        Your app's default preferences. Saved values override these
prefs_file str   No        Path to the preferences JSON file. Default: 'user_preferences.json'


Returns on success:
python{
  'success': True,
  'username': 'alice',
  'preferences': {
    'theme': 'light',        # saved value (overrides default)
    'items_per_page': 20,    # default value (not yet changed by user)
    'default_filter': 'All'  # default value
  }
}

Returns on failure:
python{'success': False, 'error': 'Username is required.'}

Examples:
import User_Preferences

# --- GameVault example ---
MY_DEFAULTS = {'theme': 'dark', 'default_filter': 'All', 'items_per_page': 20}
result = User_Preferences.get_preferences('alice', defaults=MY_DEFAULTS)
prefs = result['preferences']
print(prefs['theme'])           # 'light' if alice changed it, otherwise 'dark'
print(prefs['items_per_page'])  # 20 (default, alice hasn't changed it)

# --- Recipe app example ---
MY_DEFAULTS = {'servings': 4, 'measurement_unit': 'metric', 'show_nutrition': True}
result = User_Preferences.get_preferences('bob', defaults=MY_DEFAULTS)

# --- No defaults (just retrieve whatever was saved) ---
result = User_Preferences.get_preferences('alice')


update_preference(username, key, value, valid_options=None, prefs_file=PREFS_FILE)
Saves or updates a single preference value for a user. Optionally validates the value against a list of allowed options before saving.
Parameters:
Parameter      Type  Required  Description
username       str   Yes       The user whose preference to update
key            str   Yes       The preference key to set (e.g. 'theme')
value          any   Yes       The new value to save
valid_options  dict  No        Maps keys to lists of allowed values. If the key exists here, the value is validated before saving
prefs_file     str   No        Path to the preferences JSON file


Returns on success:
python{'success': True, 'username': 'alice', 'updated': {'theme': 'light'}}

Returns on failure:
python{'success': False, 'error': "Invalid value 'purple' for 'theme'. Options: ['dark', 'light']"}
{'success': False, 'error': 'Username is required.'}

Examples:
import User_Preferences

# Update without validation (any value accepted)
User_Preferences.update_preference('alice', 'theme', 'light')
User_Preferences.update_preference('alice', 'items_per_page', 50)

# Update with validation (value must be in the allowed list)
MY_VALID = {
    'theme': ['dark', 'light'],
    'items_per_page': [10, 20, 50, 100],
    'default_filter': ['All', 'Playing', 'Completed', 'Wishlist']
}
result = User_Preferences.update_preference('alice', 'theme', 'purple', valid_options=MY_VALID)
print(result)  # {'success': False, 'error': "Invalid value 'purple' for 'theme'. Options: ['dark', 'light']"}

result = User_Preferences.update_preference('alice', 'theme', 'light', valid_options=MY_VALID)
print(result)  # {'success': True, 'username': 'alice', 'updated': {'theme': 'light'}}

# Recipe app example
User_Preferences.update_preference('bob', 'measurement_unit', 'imperial')


reset_preferences(username, prefs_file=PREFS_FILE)
Deletes all saved preferences for a user. The next call to get_preferences() will return only the defaults your app provides.
Parameters:
Parameter  Type  Required  Description
username   str   Yes       The user whose preferences to reset
prefs_file str   No        Path to the preferences JSON file


Returns on success:
python{'success': True, 'username': 'alice', 'message': 'Preferences reset to defaults.'}

Returns on failure:
python{'success': False, 'error': 'Username is required.'}

Example:
import User_Preferences

result = User_Preferences.reset_preferences('alice')
print(result['message'])  # 'Preferences reset to defaults.'
# alice's preferences file entry is now deleted; get_preferences() will return defau


delete_preference(username, key, prefs_file=PREFS_FILE)
Removes a single saved preference key for a user without touching their other preferences. Useful when your app removes a feature and you want to clean up the stored value.
Parameters:
Parameter  Type  Required  Description
username   str   Yes       The user whose preference to delete
key        str   Yes       The specific preference key to remove
prefs_file str   No        Path to the preferences JSON file


Returns on success:
python{'success': True, 'username': 'alice', 'deleted': 'theme'}

Returns on failure:
python{'success': False, 'error': "Preference 'theme' not found for user 'alice'."}

Example:
import User_Preferences

# Remove just the 'theme' setting for alice (other preferences are untouched)
result = User_Preferences.delete_preference('alice', 'theme')
if result['success']:
    print(f"Deleted '{result['deleted']}' for {result['username']}")
