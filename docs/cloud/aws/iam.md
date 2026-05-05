# Identity and Access Management (IAM)
## ¿Qué es IAM?
- Servicio web que ayuda a controlar de forma segura el acceso a los recursos de AWS.
- Con IAM, puedes administrar de forma centralizada los permisos que controlan a qué recursos de AWS pueden acceder los usuarios.
## Usuarios y grupos en IAM
- **Cuenta root / raíz** creada por defecto, no debe ser utilizada ni compartida.
- Los **usuarios** son personas dentro de tu organización y pueden ser agrupados:
	- Cada usuario de IAM puede tener permisos individuales asignados directamente. Esto permite un control granular del acceso a los recursos de AWS.
- Los **grupos** sólo contienen usuarios, no otros grupos:
	- Agrupar usuarios con requisitos de acceso similares facilita la administración de permisos y mejora la seguridad al reducir la posibilidad de errores en la asignación de permisos.
- Los usuarios no tienen que pertenecer a un grupo y el usuario puede pertenecer a varios grupos
## Buenas prácticas - Usuarios y grupos en IAM
