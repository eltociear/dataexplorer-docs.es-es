---
title: 'Funciones escalares: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las funciones escalares de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 05/27/2020
ms.openlocfilehash: ccbd01faae3e71941c1bf4542f473410753155cf
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294583"
---
# <a name="scalar-functions"></a>Funciones escalares

## <a name="binary-functions"></a>Funciones binarias

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[binary_and()](binary-andfunction.md)|Devuelve un resultado de la operación and bit a bit entre dos valores.|
|[binary_not()](binary-notfunction.md)|Devuelve una negación bit a bit del valor de entrada.|
|[binary_or()](binary-orfunction.md)|Devuelve un resultado de la operación OR bit a bit de los dos valores.|
|[binary_shift_left()](binary-shift-leftfunction.md)|Devuelve la operación de desplazamiento a la izquierda binaria en un par de números: un << n.|
|[binary_shift_right()](binary-shift-rightfunction.md)|Devuelve la operación de desplazamiento a la derecha binaria en un par de números: un >> n.|
|[binary_xor()](binary-xorfunction.md)|Devuelve un resultado de la operación XOR bit a bit de los dos valores.|
|[bitset_count_ones()](bitset-count-onesfunction.md)|Devuelve el número de bits establecidos en la representación binaria de un número.|

## <a name="conversion-functions"></a>Funciones de conversión

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[tobool()](toboolfunction.md)|Convierte la entrada en una representación booleana (con signo de 8 bits).|
|[todatetime()](todatetimefunction.md)|Convierte la entrada en un valor escalar de fecha y hora.|
|[ToDouble ()/toreal ()](todoublefunction.md)|Convierte la entrada en un valor de tipo real. (ToDouble () y toreal () son sinónimos).|
|[tostring()](tostringfunction.md)|Convierte la entrada en una representación de cadena.|
|[totimespan()](totimespanfunction.md)|Convierte la entrada en un valor escalar de TimeSpan.|

## <a name="datetimetimespan-functions"></a>Funciones DateTime/TimeSpan

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[ago()](agofunction.md)|Resta un intervalo de tiempo especificado a la hora UTC actual.|
|[datetime_add()](datetime-addfunction.md)|Calcula un nuevo valor DateTime a partir de una parte especificada multiplicada por una cantidad especificada, agregada a una fecha y hora especificadas.|
|[datetime_part()](datetime-partfunction.md)|Extrae la parte de fecha solicitada como un valor entero.|
|[datetime_diff()](datetime-difffunction.md)|Devuelve el final del año que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[dayofmonth()](dayofmonthfunction.md)|Devuelve el número entero que representa el número de día del mes determinado.|
|[dayofweek()](dayofweekfunction.md)|Devuelve el número entero de días transcurridos desde el domingo anterior, como un intervalo de tiempo.|
|[dayofyear()](dayofyearfunction.md)|Devuelve el número entero que representa el número de días del año especificado.|
|[endofday()](endofdayfunction.md)|Devuelve el final del día que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[endofmonth()](endofmonthfunction.md)|Devuelve el final del mes que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[endofweek()](endofweekfunction.md)|Devuelve el final de la semana que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[endofyear()](endofyearfunction.md)|Devuelve el final del año que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[format_datetime()](format-datetimefunction.md)|Da formato a un parámetro DateTime basado en el parámetro de modelo de formato.|
|[format_timespan()](format-timespanfunction.md)|Da formato a un parámetro Format-TimeSpan basado en el parámetro de modelo de formato.|
|[getmonth()](getmonthfunction.md)|Obtenga el número del mes (1-12) de un valor de fecha y hora.|
|[getyear()](getyearfunction.md)|Devuelve la parte del año del argumento DateTime.|
|[hourofday()](hourofdayfunction.md)|Devuelve el número entero que representa el número de hora de la fecha especificada.|
|[make_datetime()](make-datetimefunction.md)|Crea un valor escalar DateTime a partir de la fecha y hora especificadas.|
|[make_timespan()](make-timespanfunction.md)|Crea un valor escalar TimeSpan a partir del período de tiempo especificado.|
|[monthofyear()](monthofyearfunction.md)|Devuelve el número entero que representa el número de mes del año determinado.|
|[now()](nowfunction.md)|Devuelve la hora de reloj UTC actual y, opcionalmente, puede desplazarse por un intervalo de tiempo determinado.|
|[startofday()](startofdayfunction.md)|Devuelve el inicio del día que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[startofmonth()](startofmonthfunction.md)|Devuelve el inicio del mes que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[startofweek()](startofweekfunction.md)|Devuelve el inicio de la semana que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[startofyear()](startofyearfunction.md)|Devuelve el inicio del año que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[todatetime()](todatetimefunction.md)|Convierte la entrada en un valor escalar de fecha y hora.|
|[totimespan()](totimespanfunction.md)|Convierte la entrada en un valor escalar de TimeSpan.|
|[weekofyear()](weekofyearfunction.md)|Devuelve un entero que representa el número de semana.|


## <a name="dynamicarray-functions"></a>Funciones dinámicas/de matriz

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[array_concat()](arrayconcatfunction.md)|Concatena un número de matrices dinámicas en una sola matriz.|
|[array_iif()](arrayifffunction.md)|Aplica la función IIf de elemento en las matrices.|
|[array_index_of()](arrayindexoffunction.md)|Busca el elemento especificado en la matriz y devuelve su posición.|
|[array_length()](arraylengthfunction.md)|Calcula el número de elementos de una matriz dinámica.|
|[array_slice()](arrayslicefunction.md)|Extrae un segmento de una matriz dinámica.|
|[array_split()](arraysplitfunction.md)|Crea una matriz de matrices dividida a partir de la matriz de entrada.|
|[bag_keys()](bagkeysfunction.md)|Enumera todas las claves raíz de un objeto de contenedor de propiedades dinámico.|
|[pack()](packfunction.md)|Crea un objeto dinámico (contenedor de propiedades) a partir de una lista de nombres y valores.|
|[pack_all()](packallfunction.md)|Crea un objeto dinámico (contenedor de propiedades) a partir de todas las columnas de la expresión tabular.|
|[pack_array()](packarrayfunction.md)|Empaqueta todos los valores de entrada en una matriz dinámica.|
|[repeat()](repeatfunction.md)|Genera una matriz dinámica que contiene una serie de valores iguales.|
|[set_difference()](setdifferencefunction.md)|Devuelve una matriz del conjunto de todos los valores distintos que se encuentran en la primera matriz pero que no están en otras matrices.|
|[set_has_element()](sethaselementfunction.md)|Determina si la matriz especificada contiene el elemento especificado.|
|[set_intersect()](setintersectfunction.md)|Devuelve una matriz del conjunto de todos los valores distintos que se encuentran en todas las matrices.|
|[set_union()](setunionfunction.md)|Devuelve una matriz del conjunto de todos los valores distintos que se encuentran en cualquiera de las matrices proporcionadas.|
|[treepath()](treepathfunction.md)|Enumera todas las expresiones de ruta que identifican hojas en un objeto dinámico.|
|[zip()](zipfunction.md)|La función Zip acepta cualquier número de matrices dinámicas. Devuelve una matriz cuyos elementos son una matriz con los elementos de las matrices de entrada del mismo índice.|


## <a name="window-scalar-functions"></a>Funciones escalares de ventana

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[next()](nextfunction.md)|Para el conjunto de filas serializado, devuelve un valor de una columna especificada de la fila posterior según el desplazamiento.|
|[prev()](prevfunction.md)|Para el conjunto de filas serializado, devuelve un valor de una columna especificada de la fila anterior según el desplazamiento.|
|[row_cumsum()](rowcumsumfunction.md)|Calcula la suma acumulativa de una columna.|
|[row_number()](rownumberfunction.md)|Devuelve el número de una fila en el conjunto de filas serializadas: números consecutivos a partir de un índice determinado o de 1 de forma predeterminada.|

## <a name="flow-control-functions"></a>Funciones de control de flujo

|Nombre de la función            |Descripción                                             |
|-------------------------|--------------------------------------------------------|
|[toscalar()](toscalarfunction.md)|Devuelve un valor constante escalar de la expresión evaluada.|

## <a name="mathematical-functions"></a>Funciones matemáticas

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[abs()](abs-function.md)|Calcula el valor absoluto de la entrada.|
|[acos()](acosfunction.md)|Devuelve el ángulo cuyo coseno es el número especificado (la operación inversa de cos ()).|
|[asin()](asinfunction.md)|Devuelve el ángulo cuyo seno es el número especificado (la operación inversa de sin ()).|
|[atan()](atanfunction.md)|Devuelve el ángulo cuya tangente es el número especificado (la operación inversa de tan ()).|
|[atan2()](atan2function.md)|Calcula el ángulo, en radianes, entre el eje x positivo y el rayo desde el origen hasta el punto (y, x).|
|[beta_cdf()](beta-cdffunction.md)|Devuelve la función de distribución beta acumulativa estándar.|
|[beta_inv()](beta-invfunction.md)|Devuelve el inverso de la función de densidad beta acumulativa de la versión beta.|
|[beta_pdf()](beta-pdffunction.md)|Devuelve la función de versión beta de densidad de probabilidad.|
|[cos()](cosfunction.md)|Devuelve la función de coseno.|
|[cot()](cotfunction.md)|Calcula la cotangente trigonométrica del ángulo especificado, en radianes.|
|[degrees()](degreesfunction.md)|Convierte el valor de ángulo en radianes en el valor en grados, usando la fórmula degrees = (180/PI) * ángulo en radianes.|
|[exp ()](exp-function.md)|La función exponencial de base e de x, que es e elevada a la potencia x: e ^ x.|
|[exp10()](exp10-function.md)|Función exponencial de base 10 de x, que es 10 elevado a la potencia x: 10 ^ x.|
|[exp2()](exp2-function.md)|Función exponencial de base 2 de x, que es 2 elevado a la potencia x: 2 ^ x.|
|[gamma()](gammafunction.md)|Calcula la función gamma.|
|[hash()](hashfunction.md)|Devuelve un valor hash para el valor de entrada.|
|[hash_combine()](hash_combinefunction.md)|Combina dos o más valores hash.|
|[hash_many()](hash_manyfunction.md)|Devuelve un valor hash combinado de varios valores.|
|[isfinite()](isfinitefunction.md)|Devuelve si la entrada es un valor finito (no infinito ni NaN).|
|[isinf()](isinffunction.md)|Devuelve si la entrada es un valor infinito (positivo o negativo).|
|[isnan()](isnanfunction.md)|Devuelve un valor que indica si la entrada no es un número (NaN).|
|[log()](log-function.md)|Devuelve la función logarítmica natural.|
|[log10()](log10-function.md)|Devuelve la función logarítmica común (base 10).|
|[log2()](log2-function.md)|Devuelve la función logarítmica base 2.|
|[loggamma()](loggammafunction.md)|Calcula el registro del valor absoluto de la función gamma.|
|[no ()](notfunction.md)|Invierte el valor de su argumento bool.|
|[pi()](pifunction.md)|Devuelve el valor constante de PI (π).|
|[pow()](powfunction.md)|Devuelve el resultado de elevar a Power.|
|[radians()](radiansfunction.md)|Convierte el valor de ángulo en grados en un valor en radianes, usando la fórmula radianes = (PI/180) * ángulo-en-grados.|
|[Rand ()](randfunction.md)|Devuelve un número aleatorio.|
|[intervalo ()](rangefunction.md)|Genera una matriz dinámica que contiene una serie de valores equidistantemente espaciados.|
|[round()](roundfunction.md)|Devuelve el origen redondeado a la precisión especificada.|
|[sign()](signfunction.md)|Signo de una expresión numérica.|
|[sin()](sinfunction.md)|Devuelve la función sinusoidal.|
|[sqrt()](sqrtfunction.md)|Devuelve la función raíz cuadrada.|
|[tan()](tanfunction.md)|Devuelve la función Tangent.|
|[welch_test()](welch-testfunction.md)|Calcula el valor p de la [función Welch-test](https://en.wikipedia.org/wiki/Welch%27s_t-test).|


## <a name="metadata-functions"></a>Funciones de metadatos

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[column_ifexists()](columnifexists.md)|Toma un nombre de columna como una cadena y un valor predeterminado. Devuelve una referencia a la columna si existe; de lo contrario, devuelve el valor predeterminado.|
|[current_cluster_endpoint()](current-cluster-endpoint-function.md)|Devuelve el clúster actual que ejecuta la consulta.|
|[current_database()](current-database-function.md)|Devuelve el nombre de la base de datos en el ámbito.|
|[current_principal()](current-principalfunction.md)|Devuelve la entidad de seguridad actual que ejecuta esta consulta.|
|[current_principal_details()](current-principal-detailsfunction.md)|Devuelve los detalles de la entidad de seguridad que ejecuta la consulta.|
|[current_principal_is_member_of()](current-principal-ismemberoffunction.md)|Comprueba la pertenencia al grupo o la identidad principal de la entidad de seguridad actual que ejecuta la consulta.|
|[cursor_after()](cursorafterfunction.md)|Se utiliza para obtener acceso a los registros que se ingestan después del valor anterior del cursor.|
|[estimate_data_size()](estimate-data-sizefunction.md)|Devuelve un tamaño de datos estimado de las columnas seleccionadas de la expresión tabular.|
|[extent_id()](extentidfunction.md)|Devuelve un identificador único que identifica la partición de datos ("extensión") en la que reside el registro actual.|
|[extent_tags()](extenttagsfunction.md)|Devuelve una matriz dinámica con las etiquetas de la partición de datos ("extent") en la que reside el registro actual.|
|[ingestion_time()](ingestiontimefunction.md)|Recupera la columna de fecha y hora oculta $IngestionTime del registro o null.|


## <a name="rounding-functions"></a>Funciones de redondeo

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[bin()](binfunction.md)|Redondea los valores hacia abajo hasta un entero múltiplo del tamaño de un intervalo determinado.|
|[bin_at()](binatfunction.md)|Redondea los valores a un "bin" de tamaño fijo, con control sobre el punto inicial de la ubicación. (Vea también función bin).|
|[ceiling()](ceilingfunction.md)|Calcula el entero más pequeño mayor o igual que la expresión numérica especificada.|
|[Floor ()](floorfunction.md)|Redondea los valores hacia abajo hasta un entero múltiplo del tamaño de un intervalo determinado.|


## <a name="conditional-functions"></a>Funciones condicionales

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[case()](casefunction.md)|Evalúa una lista de predicados y devuelve la primera expresión de resultado cuyo predicado se cumple.|
|[Coalesce ()](coalescefunction.md)|Evalúa una lista de expresiones y devuelve la primera expresión que no es null (o no está vacía para la cadena).|
|[IIF ()/IFF ()](iiffunction.md)|Evalúa el primer argumento (el predicado) y devuelve el valor del segundo o tercer argumento, en función de si el predicado se evaluó como true (segundo) o false (tercero).|
|[max_of()](max-offunction.md)|Devuelve el valor máximo de varias expresiones numéricas evaluadas.|
|[min_of()](min-offunction.md)|Devuelve el valor mínimo de varias expresiones numéricas evaluadas.|

## <a name="series-element-wise-functions"></a>Funciones de los elementos de la serie

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[series_add()](series-addfunction.md)|Calcula la adición de dos series numéricas por elemento.|
|[series_divide()](series-dividefunction.md)|Calcula la división de elementos de dos entradas de serie numérica.|
|[series_equals()](series-equalsfunction.md)|Calcula la `==` operación de lógica de igualdad de elementos () de dos entradas de serie numéricas.|
|[series_greater()](series-greaterfunction.md)|Calcula la `>` operación de lógica mayor () de elementos de dos entradas de serie numérica.|
|[series_greater_equals()](series-greater-equalsfunction.md)|Calcula la operación lógica mayor o igual que ( `>=` ) de elementos de dos entradas numéricas de serie.|
|[series_less()](series-lessfunction.md)|Calcula la `<` operación de lógica menos () de elementos de dos entradas de serie numérica.|
|[series_less_equals()](series-less-equalsfunction.md)|Calcula la operación de lógica inferior o igual de elemento ( `<=` ) de dos entradas numéricas de serie.|
|[series_multiply()](series-multiplyfunction.md)|Calcula la multiplicación por elemento de dos entradas de serie numéricas.|
|[series_not_equals()](series-not-equalsfunction.md)|Calcula la operación de lógica no es igual a elemento ( `!=` ) de dos entradas numéricas de serie.|
|[series_subtract()](series-subtractfunction.md)|Calcula la sustracción de elementos de dos entradas de serie numérica.|

## <a name="series-processing-functions"></a>Funciones de procesamiento de series

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[series_decompose()](series-decomposefunction.md)|Realiza una descomposición de la serie en componentes.|
|[series_decompose_anomalies()](series-decompose-anomaliesfunction.md)|Busca anomalías en una serie basándose en la descomposición de la serie.|
|[series_decompose_forecast()](series-decompose-forecastfunction.md)|Previsión basada en la descomposición de la serie.|
|[series_fill_backward()](series-fill-backwardfunction.md)|Realiza una interpolación de relleno hacia atrás de los valores que faltan en una serie.|
|[series_fill_const()](series-fill-constfunction.md)|Reemplaza los valores que faltan en una serie con un valor constante especificado.|
|[series_fill_forward()](series-fill-forwardfunction.md)|Realiza la interpolación de relleno hacia delante de los valores que faltan en una serie.|
|[series_fill_linear()](series-fill-linearfunction.md)|Realiza la interpolación lineal de los valores que faltan en una serie.|
|[series_fir()](series-firfunction.md)|Aplica un filtro de respuesta finita al impulso en una serie.|
|[series_fit_2lines()](series-fit-2linesfunction.md)|Aplica dos regresiones lineales de segmentos en una serie y devuelve varias columnas.|
|[series_fit_2lines_dynamic()](series-fit-2lines-dynamicfunction.md)|Aplica dos regresiones lineales de segmentos en una serie, devolviendo un objeto dinámico.|
|[series_fit_line()](series-fit-linefunction.md)|Aplica la regresión lineal en una serie y devuelve varias columnas.|
|[series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)|Aplica la regresión lineal en una serie y devuelve un objeto dinámico.|
|[series_iir()](series-iirfunction.md)|Aplica un filtro de respuesta infinita al impulso en una serie.|
|[series_outliers()](series-outliersfunction.md)|Puntua los puntos de anomalía en una serie.|
|[series_pearson_correlation()](series-pearson-correlationfunction.md)|Calcula el coeficiente de correlación de Pearson de dos series.|
|[series_periods_detect()](series-periods-detectfunction.md)|Busca los períodos más significativos que existen en una serie temporal.|
|[series_periods_validate()](series-periods-validatefunction.md)|Comprueba si una serie temporal contiene patrones periódicos de longitud determinada.|
|[series_seasonal()](series-seasonalfunction.md)|Busca el componente estacional de la serie.|
|[series_stats()](series-statsfunction.md)|Devuelve las estadísticas de una serie en varias columnas.|
|[series_stats_dynamic()](series-stats-dynamicfunction.md)|Devuelve las estadísticas de una serie en un objeto dinámico.|

## <a name="string-functions"></a>Funciones de cadena

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[base64_encode_tostring()](base64_encode_tostringfunction.md)|Codifica una cadena como una cadena Base64.|
|[base64_decode_tostring()](base64_decode_tostringfunction.md)|Descodifica una cadena Base64 en una cadena UTF-8.|
|[base64_decode_toarray()](base64_decode_toarrayfunction.md)|Descodifica una cadena Base64 en una matriz de valores Long.|
|[contar ()](cotfunction.md)|Cuenta las apariciones de una subcadena en una cadena. Las coincidencias de cadena sin formato pueden superponerse; las coincidencias regex no.|
|[extract()](extractfunction.md)|Obtenga una coincidencia para una expresión regular a partir de una cadena de texto.|
|[extract_all()](extractallfunction.md)|Obtiene todas las coincidencias de una expresión regular de una cadena de texto.|
|[extractjson()](extractjsonfunction.md)|Obtenga un elemento especificado fuera de un texto JSON mediante una expresión de ruta.|
|[indexof()](indexoffunction.md)|La función devuelve el índice de base cero de la primera aparición de una cadena especificada dentro de la cadena de entrada.|
|[isempty()](isemptyfunction.md)|Devuelve true si el argumento es una cadena vacía o es NULL.|
|[isnotempty()](isnotemptyfunction.md)|Devuelve true si el argumento no es una cadena vacía o null.|
|[isnotnull ()](isnotnullfunction.md)|Devuelve true si el argumento no es NULL.|
|[isnull()](isnullfunction.md)|Evalúa su único argumento y devuelve un valor booleano que indica si el argumento se evalúa como un valor null.|
|[parse_csv()](parsecsvfunction.md)|Divide una cadena determinada que representa valores separados por comas y devuelve una matriz de cadenas con estos valores.|
|[parse_ipv4()](parse-ipv4function.md)|Convierte la entrada en una representación de número larga (con signo 64).|
|[parse_ipv4_mask()](parse-ipv4-maskfunction.md)|Convierte la cadena de entrada y la máscara de prefijo IP en la representación de número larga (con signo 64).|
|[parse_ipv6 ()](parse-ipv6function.md)|Convierte la cadena IPv6 o IPv4 en una representación de cadena IPv6 canónica.|
|[parse_ipv6_mask ()](parse-ipv6-maskfunction.md)|Convierte la cadena IPv6 o IPv4 y la máscara de la máscara en una representación de cadena IPv6 canónica.|
|[parse_json()](parsejsonfunction.md)|Interpreta una cadena como un valor JSON) y devuelve el valor como dinámico.|
|[parse_url()](parseurlfunction.md)|Analiza una cadena de dirección URL absoluta y devuelve un objeto dinámico que contiene todas las partes de la dirección URL.|
|[parse_urlquery()](parseurlqueryfunction.md)|Analiza una cadena de consulta de dirección URL y devuelve un objeto dinámico que contiene los parámetros de la consulta.|
|[parse_version()](parse-versionfunction.md)|Convierte la representación de la cadena de entrada de la versión en un número decimal comparable.|
|[reemplazar ()](replacefunction.md)|Reemplace todas las coincidencias de expresiones regulares por otra cadena.|
|[reverse()](reversefunction.md)|La función hace inverso a la cadena de entrada.|
|[split()](splitfunction.md)|Divide una cadena determinada según un delimitador dado y devuelve una matriz de cadenas con las subcadenas contenidas.|
|[strcat()](strcatfunction.md)|Concatena entre 1 y 64 argumentos.|
|[strcat_delim()](strcat-delimfunction.md)|Concatena entre 2 y 64 argumentos, con delimitador, proporcionado como primer argumento.|
|[strcmp()](strcmpfunction.md)|Compara dos cadenas.|
|[strlen()](strlenfunction.md)|Devuelve la longitud, en caracteres, de la cadena de entrada.|
|[strrep()](strrepfunction.md)|Repite el número de veces especificado para la cadena (valor predeterminado: 1).|
|[substring ()](substringfunction.md)|Extrae una subcadena de una cadena de origen a partir de un índice hasta el final de la cadena.|
|[ToUpper ()](toupperfunction.md)|Convierte una cadena a mayúsculas.|
|[translate()](translatefunction.md)|Reemplaza un conjunto de caracteres ("searchList") por otro juego de caracteres ("replacementList") en una cadena dada.|
|[trim()](trimfunction.md)|Quita todas las coincidencias iniciales y finales de la expresión regular especificada.|
|[trim_end()](trimendfunction.md)|Quita la coincidencia final de la expresión regular especificada.|
|[trim_start()](trimstartfunction.md)|Quita la coincidencia inicial de la expresión regular especificada.|
|[url_decode ()](urldecodefunction.md)|La función convierte la dirección URL codificada en una representación de dirección URL normal.|
|[url_encode ()](urlencodefunction.md)|La función convierte los caracteres de la dirección URL de entrada en un formato que se puede transmitir a través de Internet.|

## <a name="ipv4ipv6-functions"></a>Funciones IPv4/IPv6

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[ipv4_compare ()](ipv4-comparefunction.md)|Compara dos cadenas IPv4.|
|[ipv4_is_match ()](ipv4-is-matchfunction.md)|Coincide con dos cadenas IPv4.|
|[parse_ipv4()](parse-ipv4function.md)|Convierte la cadena de entrada en la representación de número larga (con signo 64).|
|[parse_ipv4_mask()](parse-ipv4-maskfunction.md)|Convierte la cadena de entrada y la máscara de prefijo IP en la representación de número larga (con signo 64).|
|[ipv6_compare ()](ipv6-comparefunction.md)|Compara dos cadenas IPv4 o IPv6.|
|[ipv6_is_match ()](ipv6-is-matchfunction.md)|Coincide con dos cadenas IPv4 o IPv6.|
|[parse_ipv6 ()](parse-ipv6function.md)|Convierte la cadena IPv6 o IPv4 en una representación de cadena IPv6 canónica.|
|[parse_ipv6_mask ()](parse-ipv6-maskfunction.md)|Convierte la cadena IPv6 o IPv4 y la máscara de la máscara en una representación de cadena IPv6 canónica.|

## <a name="type-functions"></a>Funciones de tipo

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[gettype()](gettypefunction.md)|Devuelve el tipo en tiempo de ejecución de su único argumento.|

## <a name="scalar-aggregation-functions"></a>Funciones de agregación escalares

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[dcount_hll()](dcount-hllfunction.md)|Calcula el DCont a partir de los resultados de HLL (generados por HLL o HLL-Merge).|
|[hll_merge()](hllmergefunction.md)|Combina los resultados de HLL (versión escalar de la versión de agregado HLL-Merge ()).|
|[percentile_tdigest()](percentile-tdigestfunction.md)|Calcula el resultado del percentil a partir de los resultados de tdigest (generados por tdigest o Merge-tdigests).|
|[percentrank_tdigest()](percentrank-tdigestfunction.md)|Calcula la clasificación porcentual de un valor en un conjunto de DataSet.|
|[rank_tdigest()](rank-tdigest.md)|Calcula el rango relativo de un valor en un conjunto.|
|[tdigest_merge()](tdigest-mergefunction.md)|Combina los resultados de tdigest (versión escalar de la versión de agregado tdigest-Merge ()).|

## <a name="geospatial-functions"></a>Funciones geoespaciales

|Nombre de la función|Descripción|
|--------------------------------------------------------------------------|--------------------------------------------------------|
|[geo_distance_2points()](geo-distance-2points-function.md)|Calcula la distancia más corta entre dos coordenadas geoespaciales de la tierra.|
|[geo_geohash_to_central_point()](geo-geohash-to-central-point-function.md)|Calcula las coordenadas geoespaciales que representan el centro de un área rectangular de geohash.|
|[geo_point_in_circle()](geo-point-in-circle-function.md)|Calcula si las coordenadas geoespaciales están dentro de un círculo en la tierra.|
|[geo_point_to_geohash()](geo-point-to-geohash-function.md)|Calcula el valor de cadena geohash para una ubicación geográfica.|