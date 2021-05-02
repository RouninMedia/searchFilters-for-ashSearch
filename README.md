# searchFilters for ashSearch
**`searchFilters`** enable `ashSearch` to include or exclude in its search any folders at any level of site folder hierarchy.

**`searchFilters`** are a core component of `ashSearch`.

____

## `Refine_Page_List()` function

The **`Refine_Page_List()`** contains both the `Process_Exclude_Folders()` and the `Process_Include_Folders()` functions.

```
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
