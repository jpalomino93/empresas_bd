# Crear Rol de Ejecución para Funciones AWS Lambda con AWS CLI

## Índice
- [1. Crear el Rol](#1-crear-el-rol)
- [2. Crear Política](#2-crear-política)
- [3. Adjuntar Política al Rol](#3-adjuntar-política-al-rol)

## 1. Crear el Rol

Para crear un rol de ejecución para las funciones Lambda, necesitamos definir una política de confianza y luego crear el rol utilizando el AWS CLI.

### Política de confianza (trust-policy.json)

Primero, crea un archivo llamado `trust-policy.json` con el siguiente contenido:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### Comando para crear el rol

Ejecuta el siguiente comando para crear el rol:
```bash
aws iam create-role --role-name empresas_bd --assume-role-policy-document file://src/policies/trust-policy.json
```

## 2. Crear Política

Ahora crearemos una política de IAM que permita a la función Lambda realizar operaciones en la tabla "Fact_Empresas" de DynamoDB.

### Política de IAM para Lambda (dynamodb-policy.json)
Crea un archivo llamado `dynamodb-policy.json` con el siguiente contenido:

```json
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "dynamodb:PutItem",
          "dynamodb:GetItem",
          "dynamodb:UpdateItem",
          "dynamodb:DeleteItem",
          "dynamodb:Scan"
        ],
        "Resource": "arn:aws:dynamodb:us-east-1:654654589924:table/GRUPO5.FACT_EMPRESAS"
      },
      {
          "Effect": "Allow",
          "Action": [
              "logs:CreateLogGroup",
              "logs:CreateLogStream",
              "logs:PutLogEvents"
          ],
          "Resource": [
              "arn:aws:logs:us-east-1:654654589924:log-group:/aws/lambda/empresas_bd:*"
          ]
      }
    ]
  }
```
### Comando para crear la politica
Ejecuta el siguiente comando para crear la política:
```bash
aws iam create-policy --policy-name LambdaDynamoDBEmpresasPolicy --policy-document file://src/policies/dynamodb-policy.json
```
Guarda el ARN de la política que se muestra en la salida del comando. Se verá algo así:
```arn:aws:iam::654654589924:policy/LambdaDynamoDBEmpresasPolicy```

## 3. Adjuntar Política al Rol
Finalmente, adjuntaremos la política creada al rol de ejecución.
### Comando para adjuntar la política
Utiliza el siguiente comando, reemplazando [ARN_DE_LA_POLÍTICA] con el ARN que guardaste en el paso anterior:
```bash
aws iam attach-role-policy --role-name LambdaProductTableAccessRole --policy-arn [ARN_DE_LA_POLÍTICA]
```
Por ejemplo:
```bash
aws iam attach-role-policy --role-name LambdaProductTableAccessRole --policy-arn arn:aws:iam::654654589924:policy/LambdaDynamoDBEmpresasPolicy
```
Con estos pasos, habrás creado un rol de ejecución para tus funciones AWS Lambda con los permisos necesarios para interactuar con una tabla específica de DynamoDB. Recuerda reemplazar los valores de ejemplo (como IDs de cuenta y nombres de recursos) con los valores correspondientes a tu configuración de AWS.

<div style="text-align: center;">
 <img src="https://vrrvzkkbli0xp3sx.public.blob.vercel-storage.com/mindmap/24945652-6d45-4ca0-873a-ab65716e719a.png" alt="Diagrama de Flujo">
</div>


# ConectividadBD

Este proyecto contiene una función AWS Lambda para la conectividad con una base de datos DynamoDB.

## Estructura del Proyecto

- **src/**: Contiene el código fuente.
  - **handlers/**: Controladores de las funciones Lambda.
  - **policies/**: Políticas de IAM.
  - **utils/**: Utilidades y funciones auxiliares.
  - **config/**: Configuraciones varias.
- **tests/**: Contiene las pruebas unitarias y de integración.


# Despliegue de Lambda

## Obtener Role:
```shell
aws iam get-role --role-name empresas_bd
```
Obtenemos el arn del rol : ```arn:aws:iam::654654589924:role/empresas_bd```

## Desplegar lambda:

Comprimimos los archivos al desplegar del lambda : 
```
Compress-Archive  src/handlers/index.js empresas_bd.zip
```

Agregamos el zipeado los siguientes archivos : 

```shell
Compress-Archive -Path node_modules -DestinationPath empresas_bd.zip -Update
Compress-Archive -Path package.json -DestinationPath empresas_bd.zip -Update
Compress-Archive -Path package-lock.json -DestinationPath empresas_bd.zip -Update
```
Desplegamos el lambda (zip) en AWS : 

```cli
aws lambda create-function `
    --function-name empresas_bd `
    --runtime nodejs20.x `
    --zip-file fileb://empresas_bd.zip `
    --handler index.handler `
    --role arn:aws:iam::654654589924:role/empresas_bd
```

### Actualizar Politicas (Opcional):
  ```cli
  aws iam list-role-policies --role-name empresas_bd 
  aws iam get-role-policy --role-name empresas_bd --policy-name LambdaDynamoDBEmpresasPolicy
  aws iam put-role-policy --role-name empresas_bd --policy-name LambdaDynamoDBEmpresasPolicy --policy-document file://src/policies/dynamodb-policy.json
  ```

### Actualizar Lambda  (Opcional):

  ```cli
  Compress-Archive -Path src/handlers/index.js -DestinationPath empresas_bd.zip -Update

  aws lambda update-function-code `
    --function-name empresas_bd `
    --zip-file fileb://empresas_bd.zip
  ```
### AWS Lambda Function Url (Opcional)

- Crear una URL de función para la función Lambda
    ```
    aws lambda create-function-url-config `
    --function-name empresas_bd `
    --auth-type NONE
  ```
  - **Output**:  
    ```json 
    {
    "FunctionUrl": "https://4gtqz64suhv2gtjetdwg7sma240sfjae.lambda-url.us-east-1.on.aws/",
    "FunctionArn": "arn:aws:lambda:us-east-1:654654589924:function:empresas_bd",
    "AuthType": "NONE",
    "CreationTime": "2024-07-25T03:16:09.951516Z"
    }
    ```

- Obtener la URL de la función : `"FunctionUrl": "https://4gtqz64suhv2gtjetdwg7sma240sfjae.lambda-url.us-east-1.on.aws/"`
  ```
  FUNCTION_URL=$(aws lambda get-function-url-config --function-name MyFunction --query FunctionUrl --output text)
  ```

## Configurar permisos para que la URL de la función sea accesible públicamente

    aws lambda add-permission `
    --function-name empresas_bd `
    --action lambda:InvokeFunctionUrl `
    --principal "*" `
    --statement-id FunctionUrlAllowPublicAccess `
    --function-url-auth-type NONE

{
    "Statement": "{\"Sid\":\"FunctionUrlAllowPublicAccess\",\"Effect\":\"Allow\",\"Principal\":\"*\",\"Action\":\"lambda:InvokeFunctionUrl\",\"Resource\":\"arn:aws:lambda:us-east-1:654654589924:function:empresas_bd\",\"Condition\":{\"StringEquals\":{\"lambda:FunctionUrlAuthType\":\"NONE\"}}}"
}

  aws lambda add-permission --function-name empresas_bd --action lambda:InvokeFunctionUrl --principal '*' --statement-id FunctionUrlAllowPublicAccess

## Mostrar la URL de la función
  echo "La URL pública de la función Lambda es: $FUNCTION_URL"

# Comandos para DynamoDB desde el CLI

## Listar Tablas disponibles 
    aws dynamodb list-tables
## Describir una tabla específica:
    aws dynamodb describe-table --table-name GRUPO5.FACT_EMPRESAS
## Escanear un tabla
    aws dynamodb scan --table-name GRUPO5.FACT_EMPRESAS
## Consultar una tabla
    aws dynamodb query --table-name product --key-condition-expression "id = :id" --expression-attribute-values '{\":id\":{\"S\":\"1\"}}'


# Pruebas Funcionales (Postman) y Pruebas Unitarias Automatizadas

## Ejecutar pruebas de postam NEWMAN
    npx newman run tests/empresas_bd.postman_collection.json
## Ejecutar pruebas unitarias JEST
    npm test


# Parameter Store

Utilizaremos el parametor store de AWS para administrar mejor los parametros de nuestra funcion :

- /dev/rucsystem/database/index-name : NRO_RUC-index
- /dev/rucsystem/database/table-name : GRUPO5.FACT_EMPRESAS

Acceder a los parametros:

```bash
aws ssm get-parameter --name "/dev/rucsystem/database/index-name" 
aws ssm get-parameter --name "/dev/rucsystem/database/table-name" 
```

Usar etiquetas de versión:

```bash
aws ssm label-parameter-version --name "/dev/rucsystem/database/index-name" --labels "Producción"
```

Obtener un parámetro por etiqueta:
```bash
aws ssm get-parameter --name "/dev/rucsystem/database/index-name:Producción"
```