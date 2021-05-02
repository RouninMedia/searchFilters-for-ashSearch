# searchFilters for ashSearch
**`searchFilters`** enable `ashSearch` to include or exclude in its search any folders at any level of site folder hierarchy.

**`searchFilters`** are a core component of `ashSearch`.

____

## Examples of searchFilters for `ashSearch`

**searchFilters** for `ashSearch` are `JSON` strings.

To illustrate the format of **searchFilters** for `ashSearch`, here are some real-world examples of **searchFilters**:

### Example 1
**searchFilters** which exclude German, Spanish, French and Russian language content and only include the index page at the root of the `/safety-data-sheets/` folder:

```json
{
  "Exclude_Folders": {
    "de": {},
    "es": {},
    "fr": {},
    "ru": {},
    "safety-data-sheets": {
      "Include_Folders": {
        "/": {}
      }
    }
  }
}
```

**Minified:** `{"Exclude_Folders":{"de":{},"es":{},"fr":{},"ru":{},"safety-data-sheets":{"Include_Folders":{"/":{}}}}}`

_____

### Example 2
**searchFilters** which include only German language content and only include the index page at the root of the `/sicherheitsdatenblätter/` folder:

```json
{
  "Include_Folders": {
    "de": {
      "Exclude_Folders": {
        "sicherheitsdatenblätter": {
          "Include_Folders": {
            "/": {}
          }
        }
      }
    }
  }
}
```

**Minified:** `{"Include_Folders":{"de":{"Exclude_Folders":{"sicherheitsdatenblätter":{"Include_Folders":{"/":{}}}}}}}`

_____

### Example 3
**searchFilters** which include only Spanish language content and only include the index page at the root of the `/hojas-de-datos-de-seguridad/` folder:

```json
{
  "Include_Folders": {
    "es": {
      "Exclude_Folders": {
        "hojas-de-datos-de-seguridad": {
          "Include_Folders": {
            "/": {}
          }
        }
      }
    }
  }
}
```

**Minified:** `{"Include_Folders":{"es":{"Exclude_Folders":{"hojas-de-datos-de-seguridad":{"Include_Folders":{"/":{}}}}}}}`

_____

## Explanation of searchFilters Formatting

 - The root level of the **searchFilters** contains a single directive, either:
   - `Include_Folders`
   - `Exclude_Folders`

 - This single directive contains a list of one or more named folders, each with its own **Exceptions Object**
  
 - Every **Exceptions Object** contains either:
   - *a single directive*
   - no directives at all
  
 - If the **Exceptions Object** contains a single directive, it is *always* the opposite of the current directive.

 - If the **Exceptions Object** is empty, the current directive (`Include` or `Exclude`) applies to the root and all subfolders of the current named folder
  
 - If the **Exceptions Object** contains an `Include_Folders` Directive then **only** the named subfolders in the immediately following list are to be **included**
 
 - If the **Exceptions Object** contains an `Exclude_Folders` Directive then **all** the named subfolders in the immediately following list are to be **excluded**  

 - Further `Exclude_Folders` Directives may be nested inside an **Exceptions Object** inside an `Include_Folders` Directive
 
 - Further `Include_Folders` Directives may be nested inside an **Exceptions Object** inside an `Exclude_Folders` Directive

 - In *either* Directive, the shorthand `/` indicates **the root** of the current named folder

 - In *either* Directive, the shorthand `*` indicates **all the subfolders** of the current named folder

____

## Invoking the `Refine_Page_List()` function

The `Refine_Page_List()` function has two required parameters:

 - `$Page_List_Array`
 - `$Search_Filters_JSON`

The value of the parameter `$Search_Filters_JSON` is either:

 - assigned directly
 - retrieved from the queryString parameter `filters` (which supersedes direct assignation)
 - set to the default value (`[]`)  if the value is neither available to retrieve from the queryString, nor assigned directly

**Example:**

```php
// VALUE ASSIGNED DIRECTLY
$Search_Filters_JSON = '{"Exclude_Folders":{"de":{},"es":{},"fr":{},"ru":{},"safety-data-sheets":{"Include_Folders":{"/":{}}}}}';

// VALUE RETRIEVED FROM QUERYSTRING PARAMETER ?filters=
if ((isset($_GET['filters'])) && (!is_null(json_decode($_GET['filters'])))) {

  $Search_Filters_JSON = $_GET['filters'];
}

// IF VALUE NOT SET, SET DEFAULT VALUE
$Search_Filters_JSON = $Search_Filters_JSON ?? '[]';

// INVOKE FUNCTION
Refine_Page_List($Page_List_Array, $Search_Filters_JSON)
```

____

## `Refine_Page_List()` function

The **`Refine_Page_List()`** contains both the `Process_Exclude_Folders()` and the `Process_Include_Folders()` functions:

```php
function Refine_Page_List($Page_List_Array, $Search_Filters_JSON) {

  $Path = $_SERVER['DOCUMENT_ROOT'].'/.assets/content/pages/';

  $Search_Filters = json_decode($Search_Filters_JSON, TRUE);


  function Process_Exclude_Folders($Page_List_Array, $Path, $Search_Filters) {

  	$Keys = array_keys($Search_Filters);

  	for ($i = 0; $i < count($Keys); $i++) {

      if ($Search_Filters[$Keys[$i]] === []) {

      	for ($j = (count($Page_List_Array) - 1); ($j + 1) > 0; $j--) {

          if ((isset($Page_List_Array[$j])) && (strpos($Page_List_Array[$j].'/', $Path) !== FALSE)) {

  	        $Page = str_replace($Path, '', $Page_List_Array[$j]);

  	        $Page_Array = ($Path === $Page_List_Array[$j].'/') ? ['/'] : explode('/', $Page);

  	        if (($Page_Array[0] === $Keys[$i]) || (($Keys[$i] === '*') && ($Page_Array[0] !== '/'))) {

  	      	  unset($Page_List_Array[$j]);

              $Page_List_Array = array_values($Page_List_Array);
  	        }
      	  }
      	}
      }

      elseif (array_key_exists('Include_Folders', $Search_Filters[$Keys[$i]])) {

      	$New_Path = $Path.$Keys[$i].'/';
      	$New_Search_Filters = $Search_Filters[$Keys[$i]]['Include_Folders'];
      	$Page_List_Array = array_values($Page_List_Array);
  	    $Page_List_Array = Process_Include_Folders($Page_List_Array, $New_Path, $New_Search_Filters);
      }
  	}
    
    return $Page_List_Array;
  }


  function Process_Include_Folders($Page_List_Array, $Path, $Search_Filters) {

  	$Keys = array_keys($Search_Filters);

  	for ($i = (count($Page_List_Array) - 1); ($i + 1) > 0; $i--) {

  	  if ((isset($Page_List_Array[$i])) && (strpos($Page_List_Array[$i].'/', $Path) !== FALSE)) {

  	    $Page = str_replace($Path, '', $Page_List_Array[$i]);

  	    $Page_Array = ($Path === $Page_List_Array[$i].'/') ? ['/'] : explode('/', $Page);

        if (!in_array($Page_Array[0], $Keys)) {
        
          if ((($Page_Array[0] === '/') && (!in_array('/', $Keys))) || (($Page_Array[0] !== '/') && (!in_array('*', $Keys)))) {

  	        unset($Page_List_Array[$i]);

  	        $Page_List_Array = array_values($Page_List_Array);
          }
        }

        elseif (array_key_exists('Exclude_Folders', $Search_Filters[$Page_Array[0]])) {

      	  $Subfolders_to_Exclude = $Search_Filters[$Page_Array[0]]['Exclude_Folders'];
      	  $Subfolders_to_Exclude_Keys = array_keys($Search_Filters[$Page_Array[0]]['Exclude_Folders']);

      	  for ($j = 0; $j < count($Subfolders_to_Exclude_Keys); $j++) {

      	  	$Exclusion_Path = $Path.$Page_Array[0].'/'.$Subfolders_to_Exclude_Keys[$j];
      	  	$Exclusion_Path = str_replace('//', '/', $Exclusion_Path);

      	  	if (((isset($Page_List_Array[$i])) && (strpos($Page_List_Array[$i], $Exclusion_Path) !== FALSE)) || ($Subfolders_to_Exclude_Keys[$j] === '*')) {
              
      	      $New_Path = $Path.$Page_Array[0].'/';
              $New_Search_Filters = $Search_Filters[$Page_Array[0]]['Exclude_Folders'];
              $Page_List_Array = array_values($Page_List_Array);
  	          $Page_List_Array = Process_Exclude_Folders($Page_List_Array, $New_Path, $New_Search_Filters);
      	    }
      	  }
        }
      }
    }
    
    return $Page_List_Array;
  }


  switch (array_keys($Search_Filters)[0]) {

    case ('Exclude_Folders') : $Page_List_Array = Process_Exclude_Folders($Page_List_Array, $Path, $Search_Filters['Exclude_Folders']); break;
    case ('Include_Folders') : $Page_List_Array = Process_Include_Folders($Page_List_Array, $Path, $Search_Filters['Include_Folders']); break;
  }

  $Page_List_Array = array_values($Page_List_Array);

  return $Page_List_Array;
}
```
____

## Background Information...

### What is `ashSearch`?
`ashSearch` is the standard Search Module in `ash`.


### What is `ash`?

`ash` is a standard library of `da3sh`-based rich web components (or *modules*) which can be deployed across all `da3sh`-parsing environments (such as `ashiva`).
