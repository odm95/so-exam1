### Examen 1
**Universidad ICESI**  
**Curso:** Sistemas Operativos  
**Docente:** Daniel Barragán C.  
**Estudiante:** Óscar Daniel Molano  
**Código:** A00267576  
**Link:** https://github.com/odm95/so-exam1/tree/A00267576/add-exam1/A00267576  
**Tema:** Comandos de Linux, Virtualización  
**Correo:** daniel.barragan at correo.icesi.edu.co

### Objetivos
* Conocer y emplear comandos de Linux para la realización de tareas administrativas
* Virtualizar un sistema operativo
* Conocer y emplear capacidades de CentOS7 para la vitualización

### Prerrequisitos
* Virtualbox o WMWare
* Máquina virtual con sistema operativo CentOS7

### Descripción
El primer parcial del curso sistemas operativos trata sobre el manejo de los comandos de Linux, virtualización y el uso de las características de CentOS7

### Actividades
1. Incluir nombre, código (5%)
2. Ortografía y redacción cuando sea necesario (5%)
3. Resuelva los siguienes retos de la página https://cmdchallenge.com y presente la solución a cada uno de ellos a través de un ejemplo práctico en CentOS7. Presente capturas de pantalla relevantes como evidencias de lo realizado (20%)  

  * sum_all_numbers  
    
  ![][1]  
    
  ![][2]  
    
  * replace_spaces_in_filenames  
    
  ![][3]  
    
  * reverse_readme  
    
  ![][4]  
    
  * remove_duplicated_lines  
    
  ![][5]  
    
  * disp_table  
    
  ![][6]  
    
4. Realice un script que cumpla las condiciones que se describen a continuación. Presente capturas de pantalla relevantes como evidencias del funcionamiento (30%)
  * El usuario gutenberg debe existir en el sistema operativo  
  
  * El script se debe ejecutar cada 5 minutos, consulte el manual de crontab  
  * EL script debe descargar un libro del proyecto https://www.gutenberg.org/ en el directorio /home/gutenberg/mybooks
  * Si ya existe un libro en el directorio mybooks, debe ser reemplazado  
    
    Creamos el usuario gutenberg en nuestro sistema operativo.
    ![][7]  
      
    Como bien lo dice la pagina de http://www.gutenberg.org/files/ el ID de los libros se encuentra acertivamente desde el ID = 10000  
    ![][8]  
      
    Usaremos el enlace http://www.gutenberg.org/files/id/id.txt para obtener el archivo txt del directorio del libro. Como ejemplo de un resultado del anterior link vamos a buscar el libro con ID = 10000  
      
    ![][9]  
      
    A continuación se muestra el script getbook5minutes.sh que lo que hace es descargar un libro en formato .txt en un directorio especifico del usuario gutenberg, posteriormente se enseña la configuración de Crontab para que ejecute cda 5 minutos la descarga de un libro. Cabe resaltar que en cada ejecución se borran los libros anteriormente descargados.  
      
    ![][10]  
      
    Podemos observar que después de 5 minutos se actualiza el libro.  
      
    ![][11]  
      
5. Describa el funcionamiento del código fuente rickroll.c del repositorio de github https://github.com/jvns/kernel-module-fun. Muestre el funcionamiento al compilar el código y cargarlo como un módulo del kernel a través de un video que deberá cargar en Youtube e incluir el enlace en el informe (30%). Se recomienda emplear el sistema operativo Ubuntu con interfaz gráfica para esta prueba.  
  
En pocas palabras el código rickroll.c lo que hace a groso modo es modificar el system call del kernel del sistema operativo y hacer que funcione de manera diferente a lo previsto. Las acciones que realiza es cambiar la ruta de acceso a los archivos .mp3 por la ruta de la canción Never Gonna Give You Up.mp3.  
  
En el primer bloque de código podemos observas que se cargan los modulos necesarios para la compilación, la asignación de parámetros y se declara la ruta de la canción Never Gonna Give You Up.mp3 que sera reproducida cada que se intente abrir un archivo .mp3.  

```vim
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/syscalls.h>
#include <linux/string.h>


MODULE_LICENSE("GPL");
MODULE_AUTHOR("Kamal Marhubi");
MODULE_DESCRIPTION("Rickroll module");

static char *rickroll_filename = "/home/bork/media/music/Rick Astley - Never Gonna Give You Up.mp3";

module_param(rickroll_filename, charp, S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);
MODULE_PARM_DESC(rickroll_filename, "The location of the rick roll file");
```  
  
Posteriormente, como cr0 protege que el usuario realice escritura sobre el área de memoria que contiene la tabla de llamadas al sistema, se ejecutan los siguites comandos para modificar el cr0.  
  
```vim
#define DISABLE_WRITE_PROTECTION (write_cr0(read_cr0() & (~ 0x10000)))
#define ENABLE_WRITE_PROTECTION (write_cr0(read_cr0() | 0x10000))

static unsigned long **find_sys_call_table(void);
asmlinkage long rickroll_open(const char __user *filename, int flags, umode_t mode);

asmlinkage long (*original_sys_open)(const char __user *, int, umode_t);
asmlinkage unsigned long **sys_call_table;
```  
  
Luego, vemos el método rickroll_init(void) que se encarga de realizar validaciones de variables y objetos para tener un entorno de ejecución adecuado.  
  
```vim
static int __init rickroll_init(void)
{
    if(!rickroll_filename) {
	printk(KERN_ERR "No rick roll filename given.");
	return -EINVAL;  /* invalid argument */
    }

    sys_call_table = find_sys_call_table();

    if(!sys_call_table) {
	printk(KERN_ERR "Couldn't find sys_call_table.\n");
	return -EPERM;  /* operation not permitted; couldn't find general error */
    }

    /*
     * Replace the entry for open with our own function. We save the location
     * of the real sys_open so we can put it back when we're unloaded.
     */
    DISABLE_WRITE_PROTECTION;
    original_sys_open = (void *) sys_call_table[__NR_open];
    sys_call_table[__NR_open] = (unsigned long *) rickroll_open;
    ENABLE_WRITE_PROTECTION;

    printk(KERN_INFO "Never gonna give you up!\n");
    return 0;  /* zero indicates success */
}
```  
  
A continuación enontramos el método rickroll_open(const char _user *filename, int flags, umode_t mode) que se encarga de capturar los eventos donde el usuario quiere abrir archivos .mp3 y realiza el cambio de ruta por la ruta de la canción anteriormente mencionada, en caso tal que el usuario este abriendo otro tipo de archivos, se llama al método sys_open que es el método original para abrir archivos, para que de esta manera se puedan abrir normalmente.  
  
```vim  
asmlinkage long rickroll_open(const char __user *filename, int flags, umode_t mode)
{
    int len = strlen(filename);

    /* See if we should hijack the open */
    if(strcmp(filename + len - 4, ".mp3")) {
	/* Just pass through to the real sys_open if the extension isn't .mp3 */
	return (*original_sys_open)(filename, flags, mode);
    } else {
	/* Otherwise we're going to hijack the open */
	mm_segment_t old_fs;
	long fd;

	/*
	 * sys_open checks to see if the filename is a pointer to user space
	 * memory. When we're hijacking, the filename we pass will be in kernel
	 * memory. To get around this, we juggle some segment registers. I
	 * believe fs is the segment used for user space, and we're temporarily
	 * changing it to be the segment the kernel uses.
	 *
	 * An alternative would be to use read_from_user() and copy_to_user()
	 * and place the rickroll filename at the location the user code passed
	 * in, saving and restoring the memory we overwrite.
	 */
	old_fs = get_fs();
	set_fs(KERNEL_DS);

	/* Open the rickroll file instead */
	fd = (*original_sys_open)(rickroll_filename, flags, mode);

	/* Restore fs to its original value */
	set_fs(old_fs);

	return fd;
    }
}
```  
  
En seguida podemos observar otro método llamado rickroll_cleanup(void) que se encarga de restaurar el método sys_open como predeterminado.  
  
```vim  
static void __exit rickroll_cleanup(void)
{
    printk(KERN_INFO "Ok, now we're gonna give you up. Sorry.\n");

    /* Restore the original sys_open in the table */
    DISABLE_WRITE_PROTECTION;
    sys_call_table[__NR_open] = (unsigned long *) original_sys_open;
    ENABLE_WRITE_PROTECTION;
}
```  
  
Fianlmente, encontramos el método find_sys_call_table() que busca el espacio de memoria en el kernel dónde se encuentra la tabla de llamados al sistema, como podemos observar este método es llamado en el rickroll_init.  
  
```vim  
static unsigned long **find_sys_call_table() {
    unsigned long offset;
    unsigned long **sct;

    for(offset = PAGE_OFFSET; offset < ULLONG_MAX; offset += sizeof(void *)) {
	sct = (unsigned long **) offset;

	if(sct[__NR_close] == (unsigned long *) sys_close)
	    return sct;
    }

    /*
     * Given the loop limit, it's somewhat unlikely we'll get here. I don't
     * even know if we can attempt to fetch such high addresses from memory,
     * and even if you can, it will take a while!
     */
    return NULL;
}
```  
  

6. El informe debe ser entregado en formato pdf a través del moodle y el informe en formato README.md debe ser subido a un repositorio de github. El repositorio de github debe ser un fork de https://github.com/ICESI-Training/so-exam1 y para la entrega deberá hacer un Pull Request (PR) respetando la estructura definida. El código fuente y la url de github deben incluirse en el informe (10%)  

### Referencias
* https://cmdchallenge.com  
* https://www.gutenberg.org  
* https://github.com/jvns/kernel-module-fun/blob/master/rickroll.c
* https://www.youtube.com/watch?v=efEZZZf_nTc
* https://www.creatigon.com
* http://www.codigomaestro.com/linux/agregar-usuarios-y-grupos-en-centos-rhel/
* http://www.desarrollolibre.net/blog/tema/106/linux/ejecutar-script-automaticamente-con-cron-en-linux#.WdlD7mjWyMo

[1]: imagenes/sum-me1.jpg  
[2]: imagenes/sum-me2.jpg  
[3]: imagenes/replace-space.jpg  
[4]: imagenes/reverse-readme.jpg  
[5]: imagenes/remove-duplicate.jpg  
[6]: imagenes/display-table.jpg  
[7]: imagenes/create-user.jpg  
[8]: imagenes/gutenberg-evidence.jpg  
[9]: imagenes/gutenberg-evidence2.jpg  
[10]: imagenes/crontab.jpg  
[11]: imagenes/crontab-evidence.jpg  
