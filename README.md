# Convert a Google Tag Manager API repsonse into import JSON format
Follow the steps an examples in this repo to convert a Google Tag Manager [API Versions resource](https://developers.google.com/tag-platform/tag-manager/api/v2/reference/accounts/containers/versions/get) into the format required by [GTM's built-in Import/Export functionality](https://support.google.com/tagmanager/answer/6106997?hl=en).

## Steps to take
Translating the API Versions resource to the import JSON format takes the following steps:
1. Create a new object with `"exportFormatVersion": 2` and `"containerVersion": {}` key-value pairs.
2. Nest the **entire** API Versions resource under `"containerVersion"`.
3. Convert the following values from **camelCase** to **SNAKE_CASE_IN_CAPS**:
    1. All `type` values **unless** the `type` key is in the root of a `client`, `tag` or a `variable` resource.
    2. `container.containerVersion.usageContext`
    2. `tag.tagFiringOption`
    3. `consentSettings.consentStatus`
4. Copy the resulting JS object into a new JSON file and import into Google Tag Manager.

## Example in JavaScript
Here's JavaScript code for the translation. 

```
const exportJson = {
    exportFormatVersion: 2,
    containerVersion: {...}, // Paste the entire API Version resource under this key
};

// Utility to convert camelCase into SNAKE_CASE_IN_CAPS
const camelToSnake = (val) => val
    .replace(/([A-Z])/g, ' $1' )
    .split(' ')
    .join('_')
    .toUpperCase();

// Recursion to change all the required values from camelCase to SNAKE_CASE_IN_CAPS
const recursivelyChangeKeys = (parentKey, obj) => {
    if (obj.hasOwnProperty('usageContext')) {
        obj.usageContext[0] = camelToSnake(obj.usageContext[0]);
    }
    if (obj.hasOwnProperty('type') && parentKey !== 'client' && parentKey !== 'tag' && parentKey !== 'variable') {
        obj.type = camelToSnake(obj.type);
    }
    if (obj.hasOwnProperty('tagFiringOption')) {
        obj.tagFiringOption = camelToSnake(obj.tagFiringOption);
    }
    if (obj.hasOwnProperty('consentStatus')) {
        obj.consentStatus = camelToSnake(obj.consentStatus);
    }
    for (let prop in obj) {
        if (Array.isArray(obj[prop])) {
            obj[prop].forEach(item => recursivelyChangeKeys(prop, item));
        } else if (typeof obj[prop] === 'object') {
            recursivelyChangeKeys(prop, obj[prop]);
        }
    }
};

// Build the export JSON
recursivelyChangeKeys(null, exportJson.containerVersion);

// Copy to clipboard when in the JS Console, for example
copy(exportJson);
```

## Notes
This has been tested with a Web Container version and a Server Container version.

There might be edge cases not covered by this script. Please submit your findings as Issues to this repo.

For some reason, when you import a container version, the diff often claims that the resources are different and require a delete/create (when overwriting) or rename (when merging) operation for the imported assets. From what I can tell, there's no difference between what was deleted and what was created, and this behavior happens also when importing a normal export JSON rather than one created from the API resource.

The above behavior should not impact the integrity of the imported container version, even if it creates some clutter in the UI with all the unnecessary version modification log entries.
