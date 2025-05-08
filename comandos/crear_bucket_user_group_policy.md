## Objetivo
En este documento se recogen los comandos necesarios para hacer las siguientes operaciones:
- Configurar un alias para la gestión de la conexión a minio.
- Crear un bucket con versionado y retención.
- Crear una policy de control de acceso al bucket.
- Crear un usuario.
- Crear un grupo y asociarle el usuario y la policy.

## Comandos
- Crear alias para gestionar el acceso a este minio y las credenciales: SE DEBE USAR EL PUERTO DE API, NO EL DEL PORTAL
```bash
$ mc alias set tbl http://xxx.xxx.xxx.xxx:9000 admin_minio XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

- Crear bucket (se tiene que incluir la opción **--with-lock** para poder habilitar la retención, no se puede habilitar después de haber creado el bucket):
```bash
$ mc mb --with-lock tbl/test-01
Bucket created successfully `tbl/test-01`.  
```
- Habilitar versionado en el bucket
```bash
$ mc version enable tbl/test-01
tbl/test-01 versioning is enabled
```

- Añadir cuota:
```bash
$ mc quota set tbl/test-01 --size 10GB
Successfully set bucket quota of 9.3 GiB on `test-01`
```

- Añadir retención y modo governance en el bucket (se incluye la opción **--default** para que se apliquen a todos los objetos del bucket por defecto):
```bash
$ mc retention set --default governance 7d tbl/test-01/ 
```

- Para crear la policy, se debe crear un archivo con el JSON que describe la política que se quiere aplicar. Un ejmplo de política:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::test-01"
    },
    {
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::test-01/*"
    }
  ]
}

```
- Crear la policy:
```bash
$ mc admin policy create tbl test-01-policy minio_policies/test001_policy.json
Created policy `test-01-policy` successfully.
```

- Crear el usuario, se debe incluir nombre de usuario (Access Key) y la contraseña (Secret Key):
```bash
$  mc admin user add tbl
Enter Access Key: test_user
Enter Secret Key: 
Added user `test_user` successfully.
```

- Se crea el grupo y se le asocia el usuario:
```bash
$ mc admin group add tbl test001 test_user
Added members `test_user` to group `test001` successfully.
```
- Se asigna la policy al grupo:
```bash
$ mc admin policy attach tbl test-01-policy --group test001
Attached Policies: [test-01-policy]
To Group: test001
```

## Validar por comando

- Ver buckets creados
```bash
$ mc mb tbl list
Bucket created successfully `tbl`.
Bucket created successfully `list`.
```

- Listar buckets:
```bash
$ mc ls tbl
[2025-04-15 17:54:04 UTC]     0B test-00/
[2025-05-03 18:38:18 UTC]     0B test-01/AR CON LOCK DE INICIO PARA PODER HABILITAR a RETENCIÓN
```

- Listar grupos:
```bash
$ mc admin group list tbl
test-readonly
test-readwrite
test001
```

- Listar policies:
```bash

$ mc admin policy list tbl
writeonly
consoleAdmin
diagnostics
readonly
readwrite
test-01-policy
test-readonly
test-readwrite
```

- Listar usuarios:
```bash
$ mc admin user list tbl
enabled    test1                                     
enabled    test2                                     
enabled    test_user        
```

- Ver detalle de una policy:
```bash


$ mc admin policy info tbl test-01-policy
{
 "PolicyName": "test-01-policy",
 "Policy": {
  "Version": "2012-10-17",
  "Statement": [
   {
    "Effect": "Allow",
    "Action": [
     "s3:ListBucket",
     "s3:GetBucketLocation"
    ],
    "Resource": [
     "arn:aws:s3:::test-01"
    ]
   },
   {
    "Effect": "Allow",
    "Action": [
     "s3:PutObject",
     "s3:DeleteObject",
     "s3:GetObject"
    ],
    "Resource": [
     "arn:aws:s3:::test-01/*"
    ]
   }
  ]
 },
 "CreateDate": "2025-05-03T19:07:17.184Z",
 "UpdateDate": "2025-05-03T19:26:41.606Z"
}
```
- Ver detalle de un grupo:
```bash
$ mc admin group info tbl test001
Group: test001
Status: enabled
Policy: test-01-policy
Members: test_user
```

- Ver detalle de un usuario
```bash
$ mc admin user info tbl test_user
AccessKey: test_user
Status: enabled
PolicyName: 
MemberOf: [test001]
```
