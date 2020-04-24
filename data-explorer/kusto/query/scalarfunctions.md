---
title: 'Funciones escalares: Explorador de azure Data Explorer ( Scalar Functions) Microsoft Docs'
description: En este artículo se describeS Scalar Functions en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/06/2020
ms.openlocfilehash: ef807c93f9ae9b125439f55f32a2bb1eb21d46b7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509561"
---
# <a name="scalar-functions"></a>Funciones escalares

## <a name="binary-functions"></a>Funciones binarias

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[binary_and()](binary-andfunction.md)|Devuelve un resultado de la operación bit a bit entre dos valores.|
|[binary_not()](binary-notfunction.md)|Devuelve una negación bit a bit del valor de entrada.|
|[binary_or()](binary-orfunction.md)|Devuelve un resultado de la operación bit a bit de los dos valores.|
|[binary_shift_left()](binary-shift-leftfunction.md)|Devuelve la operación de desplazamiento binario a la izquierda en un par de números: un << n.|
|[binary_shift_right()](binary-shift-rightfunction.md)|Devuelve la operación de desplazamiento binario a la derecha en un par de números: un >> n.|
|[binary_xor()](binary-xorfunction.md)|Devuelve un resultado de la operación xor bit a bit de los dos valores.|
|[bitset_count_ones()](bitset-count-onesfunction.md)|Devuelve el número de bits establecidos en la representación binaria de un número.|

## <a name="conversion-functions"></a>Funciones de conversión

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[tobool()](toboolfunction.md)|Convierte la entrada en representación booleana (8 bits firmada).|
|[todatetime()](todatetimefunction.md)|Convierte la entrada en escalar datetime.|
|[todouble()/toreal()](todoublefunction.md)|Convierte la entrada en un valor de tipo real. (todouble() y toreal() son sinónimos.)|
|[tostring()](tostringfunction.md)|Convierte la entrada en una representación de cadena.|
|[totimespan()](totimespanfunction.md)|Convierte la entrada en escalar de intervalo de tiempo.|


## <a name="datetimetimespan-functions"></a>Funciones DateTime/Timespan

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[ago()](agofunction.md)|Resta un intervalo de tiempo especificado a la hora UTC actual.|
|[datetime_add()](datetime-addfunction.md)|Calcula una nueva fecha y hora a partir de un datepart especificado multiplicado por una cantidad especificada, agregada a una fecha y hora especificada.|
|[datetime_part()](datetime-partfunction.md)|Extrae la parte de fecha solicitada como un valor entero.|
|[datetime_diff()](datetime-difffunction.md)|Devuelve el final del año que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[día de mes()](dayofmonthfunction.md)|Devuelve el número entero que representa el número de día del mes especificado.|
|[día de la semana()](dayofweekfunction.md)|Devuelve el número entero de días desde el domingo anterior, como un intervalo de tiempo.|
|[día de año()](dayofyearfunction.md)|Devuelve el número entero representa el número de día del año dado.|
|[endofday()](endofdayfunction.md)|Devuelve el final del día que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[endofmonth()](endofmonthfunction.md)|Devuelve el final del mes que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[endofweek()](endofweekfunction.md)|Devuelve el final de la semana que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[final del año()](endofyearfunction.md)|Devuelve el final del año que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[format_datetime()](format-datetimefunction.md)|Da formato a un parámetro datetime en función del parámetro de patrón de formato.|
|[format_timespan()](format-timespanfunction.md)|Da formato a un parámetro de intervalo de tiempo de formato basado en el parámetro de patrón de formato.|
|[getmonth()](getmonthfunction.md)|Obtenga el número del mes (1-12) de un valor de fecha y hora.|
|[getyear()](getyearfunction.md)|Devuelve la parte del año del argumento datetime.|
|[hora del día ()](hourofdayfunction.md)|Devuelve el número entero que representa el número de hora de la fecha dada.|
|[make_datetime()](make-datetimefunction.md)|Crea un valor escalar datetime a partir de la fecha y hora especificadas.|
|[make_timespan()](make-timespanfunction.md)|Crea un valor escalar de intervalo de tiempo a partir del período de tiempo especificado.|
|[mes de año()](monthofyearfunction.md)|Devuelve el número entero representa el número de mes del año dado.|
|[ahora()](nowfunction.md)|Devuelve la hora de reloj UTC actual, opcionalmente compensada por un intervalo de tiempo determinado.|
|[inicio()](startofdayfunction.md)|Devuelve el inicio del día que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[comienzo del mes()](startofmonthfunction.md)|Devuelve el inicio del mes que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[inicio de semana()](startofweekfunction.md)|Devuelve el inicio de la semana que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[comienzo de año()](startofyearfunction.md)|Devuelve el inicio del año que contiene la fecha, desplazada por un desplazamiento, si se proporciona.|
|[todatetime()](todatetimefunction.md)|Convierte la entrada en escalar datetime.|
|[totimespan()](totimespanfunction.md)|Convierte la entrada en escalar de intervalo de tiempo.|
|[semana de año ()](weekofyearfunction.md)|Devuelve un entero que representa el número de semana.|


## <a name="dynamicarray-functions"></a>Funciones dinámicas/de matriz

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[array_concat()](arrayconcatfunction.md)|Concatena un número de matrices dinámicas a una sola matriz.|
|[array_iif()](arrayifffunction.md)|Aplica la función iif de elemento en matrices.|
|[array_index_of()](arrayindexoffunction.md)|Busca en la matriz el elemento especificado y devuelve su posición.|
|[array_length()](arraylengthfunction.md)|Calcula el número de elementos de una matriz dinámica.|
|[array_slice()](arrayslicefunction.md)|Extrae un segmento de una matriz dinámica.|
|[array_split()](arraysplitfunction.md)|Crea una matriz de matrices divididas de la matriz de entrada.|
|[bag_keys()](bagkeysfunction.md)|Enumera todas las claves raíz de un objeto de bolsa de propiedades dinámico.|
|[pack()](packfunction.md)|Crea un objeto dinámico (bolsa de propiedades) a partir de una lista de nombres y valores.|
|[pack_all()](packallfunction.md)|Crea un objeto dinámico (bolsa de propiedades) a partir de todas las columnas de la expresión tabular.|
|[pack_array()](packarrayfunction.md)|Empaqueta todos los valores de entrada en una matriz dinámica.|
|[repetir()](repeatfunction.md)|Genera una matriz dinámica que contiene una serie de valores iguales.|
|[set_difference()](setdifferencefunction.md)|Devuelve una matriz del conjunto de todos los valores distintos que se encuentran en la primera matriz pero no en otras matrices.|
|[set_has_element()](sethaselementfunction.md)|Determina si la matriz especificada contiene el elemento especificado.|
|[set_intersect()](setintersectfunction.md)|Devuelve una matriz del conjunto de todos los valores distintos que se encuentran en todas las matrices.|
|[set_union()](setunionfunction.md)|Devuelve una matriz del conjunto de todos los valores distintos que se encuentran en cualquiera de las matrices proporcionadas.|
|[treepath()](treepathfunction.md)|Enumera todas las expresiones de ruta que identifican hojas en un objeto dinámico.|
|[zip()](zipfunction.md)|La función zip acepta cualquier número de matrices dinámicas y devuelve una matriz cuyos elementos son cada una matriz que contiene los elementos de las matrices de entrada del mismo índice.|


## <a name="window-scalar-functions"></a>Funciones escalares de ventana

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[siguiente()](nextfunction.md)|Para el conjunto de filas serializado, devuelve un valor de una columna especificada de la fila posterior según el desplazamiento.|
|[prev()](prevfunction.md)|Para el conjunto de filas serializado, devuelve un valor de una columna especificada de la fila anterior según el desplazamiento.|
|[row_cumsum()](rowcumsumfunction.md)|Calcula la suma acumulada de una columna.|
|[row_number()](rownumberfunction.md)|Devuelve el número de una fila en el conjunto de filas serializado: números consecutivos a partir de un índice determinado o de 1 de forma predeterminada.|

## <a name="flow-control-functions"></a>Funciones de control de flujo

|Nombre de la función            |Descripción                                             |
|-------------------------|--------------------------------------------------------|
|[toscalar()](toscalarfunction.md)|Devuelve un valor constante escalar de la expresión evaluada.|

## <a name="mathematical-functions"></a>Funciones matemáticas

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[abdominales()](abs-function.md)|Calcula el valor absoluto de la entrada.|
|[acos()](acosfunction.md)|Devuelve el ángulo cuyo coseno es el número especificado (la operación inversa de cos()).|
|[asin()](asinfunction.md)|Devuelve el ángulo cuyo seno es el número especificado (la operación inversa de sin()).|
|[atan()](atanfunction.md)|Devuelve el ángulo cuya tangente es el número especificado (la operación inversa de tan()).|
|[atan2()](atan2function.md)|Calcula el ángulo, en radianes, entre el eje X positivo y el rayo desde el origen hasta el punto (y, x).|
|[beta_cdf()](beta-cdffunction.md)|Devuelve la función de distribución beta acumulativa estándar.|
|[beta_inv()](beta-invfunction.md)|Devuelve la inversa de la función de densidad beta de probabilidad beta acumulada.|
|[beta_pdf()](beta-pdffunction.md)|Devuelve la función beta de densidad de probabilidad.|
|[cos()](cosfunction.md)|Devuelve la función coseno.|
|[cuna()](cotfunction.md)|Calcula la cotangente trigonométrica del ángulo especificado, en radianes.|
|[grados ()](degreesfunction.md)|Convierte el valor del ángulo en radianes en valor en grados, utilizando grados de fórmula (180 / PI ) * ángulo en radios.|
|[exp()](exp-function.md)|La función exponencial base-e de x, que se eleva a la potencia x: e x.|
|[exp10()](exp10-function.md)|La función exponencial base-10 de x, que es 10 elevada a la potencia x: 10 x.|
|[exp2()](exp2-function.md)|La función exponencial base-2 de x, que es 2 elevada a la potencia x: 2 x.|
|[gamma()](gammafunction.md)|Calcula la función gamma.|
|[hash()](hashfunction.md)|Devuelve un valor hash para el valor de entrada.|
|[hash_combine()](hash_combinefunction.md)|Combina dos o más valores hash.|
|[hash_many()](hash_manyfunction.md)|Devuelve un valor hash combinado de varios valores.|
|[isfinite()](isfinitefunction.md)|Devuelve si la entrada es un valor finito (no es infinito ni NaN).|
|[isinf()](isinffunction.md)|Devuelve si la entrada es un valor infinito (positivo o negativo).|
|[isnan()](isnanfunction.md)|Devuelve si la entrada es el valor Not-a-Number (NaN).|
|[log()](log-function.md)|Devuelve la función de logarithm natural.|
|[log10()](log10-function.md)|Retuns la función de laritmo comon (base-10).|
|[log2()](log2-function.md)|Devuelve la función de logarithm base-2.|
|[loggamma()](loggammafunction.md)|Calcula el registro del valor absoluto de la función gamma.|
|[no()](notfunction.md)|Invierte el valor de su argumento bool.|
|[pi()](pifunction.md)|Devuelve el valor constante de Pi (o).|
|[pow()](powfunction.md)|Devuelve el resultado de elevar a la energía.|
|[radianes()](radiansfunction.md)|Convierte el valor del ángulo en grados en valor en radianes, utilizando radianes de fórmula (PI / 180 ) * ángulo en grados.|
|[rand()](randfunction.md)|Devuelve un número aleatorio.|
|[rango()](rangefunction.md)|Genera una matriz dinámica que contiene una serie de valores igualmente espaciados.|
|[redondo()](roundfunction.md)|Devuelve el origen redondeado a la precisión especificada.|
|[sign()](signfunction.md)|Signo de una expresión numérica.|
|[pecado()](sinfunction.md)|Devuelve la función sinusoito.|
|[sqrt()](sqrtfunction.md)|Devuelve la función de raíz cuadrada.|
|[bronceado()](tanfunction.md)|Devuelve la función tangente.|
|[welch_test()](welch-testfunction.md)|Calcula el valor p de la [función Welch-test](https://en.wikipedia.org/wiki/Welch%27s_t-test).|


## <a name="metadata-functions"></a>Funciones de metadatos

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[column_ifexists()](columnifexists.md)|Toma un nombre de columna como una cadena y un valor predeterminado. Devuelve una referencia a la columna si existe, de lo contrario - devuelve el valor predeterminado.|
|[current_cluster_endpoint()](current-cluster-endpoint-function.md)|Devuelve el clúster actual que ejecuta la consulta.|
|[current_database()](current-database-function.md)|Devuelve el nombre de la base de datos en el ámbito.|
|[current_principal()](current-principalfunction.md)|Devuelve la entidad de seguridad actual que ejecuta esta consulta.|
|[current_principal_details()](current-principal-detailsfunction.md)|Devuelve los detalles de la entidad de seguridad que ejecuta la consulta.|
|[current_principal_is_member_of()](current-principal-ismemberoffunction.md)|Comprueba la pertenencia al grupo o la identidad principal de la entidad de seguridad actual que ejecuta la consulta.|
|[cursor_after()](cursorafterfunction.md)|Se utiliza para acceder a los registros que se ingirieron después del valor anterior del cursor.|
|[estimate_data_size()](estimate-data-sizefunction.md)|Devuelve un tamaño de datos estimado de las columnas seleccionadas de la expresión tabular.|
|[extent_id()](extentidfunction.md)|Devuelve un identificador único que identifica la partición de datos ("extensión") en la que reside el registro actual.|
|[extent_tags()](extenttagsfunction.md)|Devuelve una matriz dinámica con las etiquetas de la partición de datos ("extensión") en la que reside el registro actual.|
|[ingestion_time()](ingestiontimefunction.md)|Recupera la $IngestionTime columna datetime oculta del registro o null.|


## <a name="rounding-functions"></a>Funciones de redondeo

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[bin()](binfunction.md)|Redondea los valores hacia abajo hasta un entero múltiplo del tamaño de un intervalo determinado.|
|[bin_at()](binatfunction.md)|Redondea los valores hacia abajo a una "bin" de tamaño fijo, con control sobre el punto inicial de la ubicación. (Véase también función bin.)|
|[techo()](ceilingfunction.md)|Calcula el entero más pequeño mayor o igual que la expresión numérica especificada.|
|[piso()](floorfunction.md)|Redondea los valores hacia abajo hasta un entero múltiplo del tamaño de un intervalo determinado.|


## <a name="conditional-functions"></a>Funciones condicionales

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[caso()](casefunction.md)|Evalúa una lista de predicados y devuelve la primera expresión de resultado cuyo predicado se cumple.|
|[coalesce()](coalescefunction.md)|Evalúa una lista de expresiones y devuelve la primera expresión no nula (o no vacía para cadena).|
|[iif()/iff()](iiffunction.md)|Evalúa el primer argumento (el predicado) y devuelve el valor del segundo o tercer argumento, dependiendo de si el predicado se evaluó como true (segundo) o false (tercero).|
|[max_of()](max-offunction.md)|Devuelve el valor máximo de varias expresiones numéricas evaluadas.|
|[min_of()](min-offunction.md)|Devuelve el valor mínimo de varias expresiones numéricas evaluadas.|

## <a name="series-element-wise-functions"></a>Funciones de elementos de la serie

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[series_add()](series-addfunction.md)|Calcula la adición de elementos de dos entradas de serie numérica.|
|[series_divide()](series-dividefunction.md)|Calcula la división de elementos de dos entradas de serie numérica.|
|[series_equals()](series-equalsfunction.md)|Calcula la operación lógica`==`igual a elemento sin ( ) de dos entradas de serie numérica.|
|[series_greater()](series-greaterfunction.md)|Calcula la operación lógica`>`mayor de elementos ( ) de dos entradas de serie numérica.|
|[series_greater_equals()](series-greater-equalsfunction.md)|Calcula la operación lógica mayor`>=`o igual a un elemento mayor o igual ( ) de dos entradas de serie numérica.|
|[series_less()](series-lessfunction.md)|Calcula la operación lógica`<`de elemento menos ( ) de dos entradas de serie numérica.|
|[series_less_equals()](series-less-equalsfunction.md)|Calcula la operación lógica de`<=`elemento sin igual o igual ( ) de dos entradas de serie numérica.|
|[series_multiply()](series-multiplyfunction.md)|Calcula la multiplicación por elementos de dos entradas de serie numérica.|
|[series_not_equals()](series-not-equalsfunction.md)|Calcula la operación lógica no`!=`es igual a ( ) por elementos de dos entradas de serie numérica.|
|[series_subtract()](series-subtractfunction.md)|Calcula la resta de elementos de dos entradas de serie numérica.|

## <a name="series-processing-functions"></a>Funciones de procesamiento de series

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[series_decompose()](series-decomposefunction.md)|Realiza una descomposición de la serie en componentes.|
|[series_decompose_anomalies()](series-decompose-anomaliesfunction.md)|Encuentra anomalías en una serie basada en la descomposición de la serie.|
|[series_decompose_forecast()](series-decompose-forecastfunction.md)|Pronóstico basado en la descomposición de la serie.|
|[series_fill_backward()](series-fill-backwardfunction.md)|Realiza la interpolación de relleno hacia atrás de los valores que faltan en una serie.|
|[series_fill_const()](series-fill-constfunction.md)|Reemplaza los valores que faltan en una serie por un valor constante especificado.|
|[series_fill_forward()](series-fill-forwardfunction.md)|Realiza la interpolación de relleno hacia delante de los valores que faltan en una serie.|
|[series_fill_linear()](series-fill-linearfunction.md)|Realiza la interpolación lineal de los valores que faltan en una serie.|
|[series_fir()](series-firfunction.md)|Aplica un filtro de respuesta de impulso finito en una serie.|
|[series_fit_2lines()](series-fit-2linesfunction.md)|Aplica dos segmentos de regresión lineal en una serie, devolviendo varias columnas.|
|[series_fit_2lines_dynamic()](series-fit-2lines-dynamicfunction.md)|Aplica dos segmentos de regresión lineal en una serie, devolviendo el objeto dinámico.|
|[series_fit_line()](series-fit-linefunction.md)|Aplica regresión lineal en una serie, devolviendo varias columnas.|
|[series_fit_line_dynamic()](series-fit-line-dynamicfunction.md)|Aplica la regresión lineal en una serie, devolviendo el objeto dinámico.|
|[series_iir()](series-iirfunction.md)|Aplica un filtro De respuesta de impulso infinito en una serie.|
|[series_outliers()](series-outliersfunction.md)|Anota puntos de anomalía en una serie.|
|[series_pearson_correlation()](series-pearson-correlationfunction.md)|Calcula el coeficiente de correlación de Pearson de dos series.|
|[series_periods_detect()](series-periods-detectfunction.md)|Encuentra los períodos más significativos que existen en una serie temporal.|
|[series_periods_validate()](series-periods-validatefunction.md)|Comprueba si una serie temporal contiene patrones periódicos de longitudes determinadas.|
|[series_seasonal()](series-seasonalfunction.md)|Encuentra el componente estacional de la serie.|
|[series_stats()](series-statsfunction.md)|Devuelve estadísticas de una serie en varias columnas.|
|[series_stats_dynamic()](series-stats-dynamicfunction.md)|Devuelve estadísticas de una serie en objeto dinámico.|

## <a name="string-functions"></a>Funciones de cadena

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[base64_encode_tostring()](base64_encode_tostringfunction.md)|Codifica una cadena como cadena base64.|
|[base64_decode_tostring()](base64_decode_tostringfunction.md)|Descodifica una cadena base64 en una cadena UTF-8.|
|[base64_decode_toarray()](base64_decode_toarrayfunction.md)|Descodifica una cadena base64 en una matriz de valores largos.|
|[countof()](cotfunction.md)|Cuenta las apariciones de una subcadena en una cadena. Las coincidencias de cadena sin formato se pueden superponer; las coincidencias de expresión regular no lo hacen.|
|[extracto()](extractfunction.md)|Obtenga una coincidencia para una expresión regular a partir de una cadena de texto.|
|[extract_all()](extractallfunction.md)|Obtener todas las coincidencias de una expresión regular de una cadena de texto.|
|[extractjson()](extractjsonfunction.md)|Obtenga un elemento especificado fuera de un texto JSON mediante una expresión de ruta.|
|[indexof()](indexoffunction.md)|Función notifica el índice de base cero de la primera aparición de una cadena especificada dentro de la cadena de entrada.|
|[isempty()](isemptyfunction.md)|Devuelve true si el argumento es una cadena vacía o es null.|
|[isnotempty()](isnotemptyfunction.md)|Devuelve true si el argumento no es una cadena vacía ni es un valor null.|
|[isnotnull()](isnotnullfunction.md)|Devuelve true si el argumento no es null.|
|[isnull()](isnullfunction.md)|Evalúa su único argumento y devuelve un valor bool que indica si el argumento se evalúa como un valor nulo.|
|[parse_csv()](parsecsvfunction.md)|Divide una cadena determinada que representa valores separados por comas y devuelve una matriz de cadenas con estos valores.|
|[parse_ipv4()](parse-ipv4function.md)|Convierte la entrada en una representación de número larga (firmada de 64 bits).|
|[parse_json()](parsejsonfunction.md)|Interpreta una cadena como un valor JSON) y devuelve el valor como dinámico.|
|[parse_url()](parseurlfunction.md)|Analiza una cadena de dirección URL absoluta y devuelve un objeto dinámico que contiene todas las partes de la dirección URL.|
|[parse_urlquery()](parseurlqueryfunction.md)|Analiza una cadena de consulta url y devuelve un objeto dinámico que contiene los parámetros Query.|
|[parse_version()](parse-versionfunction.md)|Convierte la representación de cadena de entrada de la versión en un número decimal comparable.|
|[reemplazar()](replacefunction.md)|Reemplace todas las coincidencias de expresiones regulares por otra cadena.|
|[inverso()](reversefunction.md)|La función hace que se revierta la cadena de entrada.|
|[split()](splitfunction.md)|Divide una cadena determinada según un delimitador determinado y devuelve una matriz de cadenas con las subcadenas contenidas.|
|[strcat()](strcatfunction.md)|Concatena entre 1 y 64 argumentos.|
|[strcat_delim()](strcat-delimfunction.md)|Concatena entre 2 y 64 argumentos, con delimitador, proporcionado como primer argumento.|
|[strcmp()](strcmpfunction.md)|Compara dos cadenas.|
|[strlen()](strlenfunction.md)|Devuelve la longitud, en caracteres, de la cadena de entrada.|
|[strrep()](strrepfunction.md)|Repite la cantidad de veces proporcionada por la cadena dada (predeterminado - 1).|
|[subcadena()](substringfunction.md)|Extrae una subcadena de una cadena de origen a partir de algún índice hasta el final de la cadena.|
|[toupper()](toupperfunction.md)|Convierte una cadena a mayúsculas.|
|[translate()](translatefunction.md)|Reemplaza un conjunto de caracteres ('searchList') por otro conjunto de caracteres ('replacementList') en una cadena determinada.|
|[ajuste()](trimfunction.md)|Quita todas las coincidencias iniciales y finales de la expresión regular especificada.|
|[trim_end()](trimendfunction.md)|Quita la coincidencia final de la expresión regular especificada.|
|[trim_start()](trimstartfunction.md)|Quita la coincidencia inicial de la expresión regular especificada.|
|[url_decode()](urldecodefunction.md)|La función convierte la dirección URL codificada en una representación de URL a normal.|
|[url_encode()](urlencodefunction.md)|La función convierte los caracteres de la URL de entrada en un formato que se puede transmitir a través de Internet.|

## <a name="ip-v4-functions"></a>Funciones IP v4

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[ipv4_compare()](ipv4-comparefunction.md)|Compara dos cadenas IPv4.|
|[ipv4_is_match()](ipv4-is-matchfunction.md)|Coincide con dos cadenas IPv4.|
|[parse_ipv4()](parse-ipv4function.md)|Convierte la cadena de entrada en una representación de número larga (firmada de 64 bits).|
|[parse_ipv4_mask()](parse-ipv4-maskfunction.md)|Convierte la cadena de entrada y la máscara de prefijo IP en una representación de número larga (firmada de 64 bits).|

## <a name="type-functions"></a>Funciones de tipo

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[gettype()](gettypefunction.md)|Devuelve el tipo de tiempo de ejecución de su único argumento.|


## <a name="scalar-aggregation-functions"></a>Funciones de agregación escalar

|Nombre de la función     |Descripción                                          |
|-------------------------|--------------------------------------------------------|
|[dcount_hll()](dcount-hllfunction.md)|Calcula el dcount a partir de los resultados hll (que fue generado por hll o hll-merge).|
|[hll_merge()](hllmergefunction.md)|Combina los resultados de hll (versión escalar de la versión agregada hll-merge()).|
|[percentile_tdigest()](percentile-tdigestfunction.md)|Calcula el resultado del percentil a partir de los resultados de tdigest (que se generó mediante tdigest o merge-tdigests).|
|[percentrank_tdigest()](percentrank-tdigestfunction.md)|Calcula la clasificación porcentual de un valor en un conjunto de datos.|
|[rank_tdigest()](rank-tdigest.md)|Calcula el rango relativo de un valor en un conjunto.|
|[tdigest_merge()](tdigest-mergefunction.md)|Combina los resultados de tdigest (versión escalar de la versión agregada tdigest-merge()).|

## <a name="geo-spatial-functions"></a>Funciones geoespaciales

|Nombre de la función|Descripción|
|--------------------------------------------------------------------------|--------------------------------------------------------|
|[geo_distance_2points()](geo-distance-2points-function.md)|Calcula la distancia más corta entre dos coordenadas geoespaciales en la Tierra.|
|[geo_geohash_to_central_point()](geo-geohash-to-central-point-function.md)|Calcula las coordenadas geoespaciales que representan el centro de un área rectangular de Geohash.|
|[geo_point_in_circle()](geo-point-in-circle-function.md)|Calcula si las coordenadas geoespaciales están dentro de un círculo en la Tierra.|
|[geo_point_to_geohash()](geo-point-to-geohash-function.md)|Calcula el valor de cadena Geohash para una ubicación geográfica.|