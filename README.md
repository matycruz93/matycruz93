### Hi there 👋

#include <stdio.h>      // libreria estandar
#include <stdlib.h>     // para usar exit y funciones de la libreria standard
#include <string.h>
#include <pthread.h>    // para usar threads
#include <semaphore.h>  // para usar semaforos
#include <unistd.h>


#define LIMITE 50
//declaro semaforos tipos mutex
pthread_mutex_t salarMutex;
pthread_mutex_t sartenMutex;
pthread_mutex_t cocinarMutex;
pthread_mutex_t hornearMutex;
pthread_mutex_t ganadorMutex;
//void prueba(char cadena[]);
//void leerArchivo();

FILE *ak;
int flanco=0;
int Equipo=0;


struct semaforos {
    sem_t sem_mezclar;
    sem_t sem_salar;
    sem_t sem_empanar;
    sem_t sem_sarten;
    sem_t sem_cocinarT;
    sem_t sem_hornearT;
    sem_t sem_cortarLTCPT;
    //sem_t sem_sandwichListo;
	//poner demas semaforos aqui
};

//creo los pasos con los ingredientes
struct paso {
   char accion [LIMITE];
   char ingredientes[4][LIMITE];
   
};

//creo los parametros de los hilos 
struct parametro {
 int equipo_param;
 struct semaforos semaforos_param;
 struct paso pasos_param[8];
 //FILE *ak;
};

//funcion para imprimir las acciones y los ingredientes de la accion
void* imprimirAccion(void *data, char *accionIn) {
	struct parametro *mydata = data;
	//FILE *ak;//para escribir archivo
	ak =fopen("Salida.txt","a");//para escribir archivo
	//calculo la longitud del array de pasos 
	int sizeArray = (int)( sizeof(mydata->pasos_param) / sizeof(mydata->pasos_param[0]));
	//indice para recorrer array de pasos 
	int i;
	for(i = 0; i < sizeArray; i ++){
		//pregunto si la accion del array es igual a la pasada por parametro (si es igual la funcion strcmp devuelve cero)
		if(strcmp(mydata->pasos_param[i].accion, accionIn) == 0){
		printf("\tEquipo %d - accion %s \n " , mydata->equipo_param, mydata->pasos_param[i].accion);
		fprintf(ak,"\tEquipo %d - accion %s \n",mydata->equipo_param,mydata->pasos_param[i].accion);//para escribir archivo
		//calculo la longitud del array de ingredientes
		int sizeArrayIngredientes = (int)( sizeof(mydata->pasos_param[i].ingredientes) / sizeof(mydata->pasos_param[i].ingredientes[0]) );
		//indice para recorrer array de ingredientes
		int h;
		printf("\tEquipo %d -----------ingredientes : ----------\n",mydata->equipo_param);
		fprintf(ak,"\tEquipo %d -----------ingredientes : ----------\n",mydata->equipo_param); 
			for(h = 0; h < sizeArrayIngredientes; h++) {
				//consulto si la posicion tiene valor porque no se cuantos ingredientes tengo por accion 
				if(strlen(mydata->pasos_param[i].ingredientes[h]) != 0) {
							printf("\tEquipo %d ingrediente  %d : %s \n",mydata->equipo_param,h,mydata->pasos_param[i].ingredientes[h]);
							fprintf(ak,"\tEquipo %d ingrediente %d : %s \n",mydata->equipo_param,h,mydata->pasos_param[i].ingredientes[h]);
				}
			}
		}
	}
 //fclose(ak);
}

//funcion para tomar de ejemplo
void* cortar(void *data) {
	//creo el nombre de la accion de la funcion 
	char *accion = "cortar";
	//creo el puntero para pasarle la referencia de memoria (data) del struct pasado por parametro (la cual es un puntero). 
	struct parametro *mydata = data;
	//llamo a la funcion imprimir le paso el struct y la accion de la funcion
	imprimirAccion(mydata,accion);
	//uso sleep para simular que que pasa tiempo
	usleep( 1000000 );
	//doy la señal a la siguiente accion (cortar me habilita mezclar)
    sem_post(&mydata->semaforos_param.sem_mezclar);
	
    pthread_exit(NULL);
}
void* mezclar(void *data){
	struct parametro *mydata = data;
	sem_wait(&mydata->semaforos_param.sem_mezclar);
	char *accion = "mezclar";
	imprimirAccion(mydata,accion);
	usleep(1000000);
	sem_post(&mydata->semaforos_param.sem_salar);

	pthread_exit(NULL);
}
void* salar(void *data){
	struct parametro *mydata= data;
	sem_wait(&mydata->semaforos_param.sem_salar);
	pthread_mutex_lock(&salarMutex);//bloqueo
	char *accion = "salar";
	imprimirAccion(mydata,accion);
	usleep(5000000);
	pthread_mutex_unlock(&salarMutex);//desbloquear
	sem_post(&mydata->semaforos_param.sem_empanar);
	pthread_exit(NULL);
}
void* empanarMezcla(void *data){
	struct parametro *mydata = data;
	sem_wait(&mydata->semaforos_param.sem_empanar);
	char *accion = "empanar";
	imprimirAccion(mydata,accion);
	usleep(1000000);
	sem_post(&mydata->semaforos_param.sem_sarten);
	pthread_exit(NULL);
}
void* cocinar(void *data){
	struct parametro *mydata = data;
	sem_wait(&mydata->semaforos_param.sem_sarten);
	pthread_mutex_lock(&sartenMutex);//bloqueo;
	char *accion = "sarten";
	imprimirAccion(mydata,accion);
	usleep(5000000);
	sem_post(&mydata->semaforos_param.sem_cocinarT);
	pthread_mutex_unlock(&sartenMutex);//desbloqueo
	pthread_exit(NULL);
}
void* hornear(void *data){
	struct parametro *mydata = data;
	pthread_mutex_lock(&hornearMutex);//bloqueo;
	char *accion = "hornear";
	imprimirAccion(mydata,accion);
	usleep(1000000);
	sem_post(&mydata->semaforos_param.sem_hornearT);
	pthread_mutex_unlock(&hornearMutex);//desbloqueo;
	pthread_exit(NULL);
}
void* cortarLTCP(void *data){
	struct parametro *mydata = data;
	char *accion = "cortarLTCP";
	imprimirAccion(mydata,accion);
	usleep(1000000);
	sem_post(&mydata->semaforos_param.sem_cortarLTCPT);
	pthread_exit(NULL);
}
void* armarSandwich(void *data){
	struct parametro *mydata = data;
	sem_wait(&mydata->semaforos_param.sem_cortarLTCPT);
	sem_wait(&mydata->semaforos_param.sem_hornearT);
	sem_wait(&mydata->semaforos_param.sem_cocinarT);
	pthread_mutex_lock(&ganadorMutex);
	if(flanco==0){
	char *accion = "GANADOR";
	imprimirAccion(mydata,accion);
	Equipo=mydata->equipo_param;
	printf("\tEL GANADOR ES EL EQUIPO:%d\n",Equipo);
	ak=fopen("Salida.txt","a");
	fprintf(ak,"\tEL GANADOR ES EL EQUIPO:%d\n",Equipo);
	flanco = 1;
	}
	usleep(1000000);
	pthread_mutex_unlock(&ganadorMutex);
	pthread_exit(NULL);
}

void* ejecutarReceta(void *i) {
	
	//variables semaforos
	sem_t sem_mezclar;
	sem_t sem_salar;
	sem_t sem_sarten;
	sem_t sem_empanar;
	sem_t sem_cocinarT;
	sem_t sem_hornearT;
	sem_t sem_cortarLTCPT;
	//sem_t sem_hamLista;
	//crear variables semaforos aqui
	
	//variables hilos
	pthread_t p1;
	pthread_t p2;
	pthread_t p3;
	pthread_t p4;
	pthread_t p5;
	pthread_t p6;
	pthread_t p7;
	 
	//crear variables hilos aqui
	
	//numero del equipo (casteo el puntero a un int)
	int p = *((int *) i);
	
	printf("Ejecutando equipo %d \n", p);

	//reservo memoria para el struct
	struct parametro *pthread_data = malloc(sizeof(struct parametro));

	//seteo los valores al struct
	
	//seteo numero de grupo
	pthread_data->equipo_param = p;

	//seteo semaforos
	pthread_data->semaforos_param.sem_mezclar = sem_mezclar;
	pthread_data->semaforos_param.sem_salar = sem_salar;
	pthread_data->semaforos_param.sem_empanar = sem_empanar;
	pthread_data->semaforos_param.sem_sarten= sem_sarten;
	pthread_data->semaforos_param.sem_cocinarT = sem_cocinarT;
	pthread_data->semaforos_param.sem_hornearT = sem_hornearT;
	pthread_data->semaforos_param.sem_cortarLTCPT =sem_cortarLTCPT;
	//pthread_data->semaforos_param.sem_sandwichListo =sem_sandwichListo;
	//setear demas semaforos al struct aqui

FILE *fp;
char* archivo="Receta.txt";
char * palabra =NULL;
size_t len=0;
ssize_t leer;

fp=fopen(archivo,"r");
if(fp == NULL)
	exit(-1);
int k=0;

while((leer =getline(&palabra,&len,fp))!=-1){

int j=0;
char *cadena = strtok(palabra, "|");

while(cadena !=NULL)
{

	if(j ==0){
	//j=0;
	strcpy(pthread_data->pasos_param[k].accion,cadena);
	//printf("Accion's'\n",cadena);
	//printf(" %d\n",i)
	cadena= strtok(NULL,"|");
	}
	else{
	strcpy(pthread_data->pasos_param[k].ingredientes[j],cadena);
        //printf("Ingrediente'%s'\n",cadena);
	//printf(" %d\n","i");
	cadena =strtok(NULL,"|");
        }
	j++;
}
	k++;
}
fclose(fp);
//char* palabra=NULL;
//int j=0;
//int k=0;
		//while(feof(fp)==0){
		//fgets(palabra,100,fp);
		//while(fgets(palabra,100,fp)){
		//  while((read =getline(&palabra,&len,fp))!=-1){
		  //      int init_size=strlen(palabra);	
		//	char *cadena =strtok(palabra,"|");
		//	while(cadena !=NULL){
		//	  if(j==0)
		//	{
		//	  strcpy(pthread_data->pasos_param[k].accion,cadena);
			 //strcpy(
			 // printf("Accion '%s'\n",cadena);
			  //printf("valor de k: %d\n",k);
		//	  cadena=strtok(NULL,"|");
		//	}
		//	else{
		//		strcpy(pthread_data->pasos_param[k].ingredientes[j],cadena);
				//printf("Ingredientes '%s'\n",cadena);
				//printf("valor de j:%d\n",j);
		//		cadena=strtok(NULL,"|");
				
                        //   }
		//	    j++;
		//	}
			
		//	k++;
//}
//fclose(fp);			//printf(palabra);
//		strcpy(pthread_data->pasos_param[0].accion,cadena);
//		printf(pthread_data->pasos_param[0].accion);
//		strcpy(pthread_data->pasos_param[0].ingredientes[0],"palabra");
//		strcpy(pthread_data->pasos_param[0].ingredientes[0],"palabra");
//		strcpy(pthread_data->pasos_param[0].ingredientes[0],"palabra");
	
//		strcpy(pthread_data->pasos_param[1].accion,"palabra");
//		strcpy(pthread_data->pasos_param[1].ingredientes[0],"palabra");
//		strcpy(pthread_data->pasos_param[1].ingredientes[1],"palabra");
//		strcpy(pthread_data->pasos_param[1].ingredientes[2],"palabra");
//		strcpy(pthread_data->pasos_param[1].ingredientes[3],"palabra");

//		strcpy(pthread_data->pasos_param[2].accion,"palabra");
//		strcpy(pthread_data->pasos_param[2].ingredientes[0],"palabra");

//		strcpy(pthread_data->pasos_param[k].accion,palabra);
//		strcpy(pthread_data->pasos_param[3].ingredientes[0],"palabra");
//		strcpy(pthread_data->pasos_param[3].ingredientes[1],"palabra");
	
//		strcpy(pthread_data->pasos_param[k].accion,"palabra");
//		strcpy(pthread_data->pasos_param[4].ingredientes[0],"palabra");

//		strcpy(pthread_data->pasos_param[k].accion,"palabra");
//		strcpy(pthread_data->pasos_param[5].ingredientes[0],"palabra");

//		strcpy(pthread_data->pasos_param[k].accion,"palabra");
//		strcpy(pthread_data->pasos_param[6].ingredientes[0],"palabra");
//		strcpy(pthread_data->pasos_param[6].ingredientes[1],"palabra");

//		strcpy(pthread_data->pasos_param[k].accion,"palabra");
//		i++;
//}
 
		//token =strtok(NULL,"|");
		//}
	//}
//}
	//seteo las acciones y los ingredientes (Faltan acciones e ingredientes) ¿Se ve hardcodeado no? ¿Les parece bien?
 //    	strcpy(pthread_data->pasos_param[0].accion, "cortar");
//	strcpy(pthread_data->pasos_param[0].ingredientes[0], "ajo");
  //      strcpy(pthread_data->pasos_param[0].ingredientes[1], "perejil");

//	strcpy(pthread_data->pasos_param[1].accion, "mezclar");
//	strcpy(pthread_data->pasos_param[1].ingredientes[0], "ajo");
  //      strcpy(pthread_data->pasos_param[1].ingredientes[1], "perejil");
 //	strcpy(pthread_data->pasos_param[1].ingredientes[2], "huevo");
//	strcpy(pthread_data->pasos_param[1].ingredientes[3], "carne");
//
//	strcpy(pthread_data->pasos_param[2].accion, "salar");
//	strcpy(pthread_data->pasos_param[2].ingredientes[0],"mezcla");
	
//	strcpy(pthread_data->pasos_param[3].accion, "empanar");
//	strcpy(pthread_data->pasos_param[3].ingredientes[0],"mezcla");
//	strcpy(pthread_data->pasos_param[3].ingredientes[1],"sal");
	
//	strcpy(pthread_data->pasos_param[4].accion, "sarten");
//	strcpy(pthread_data->pasos_param[4].ingredientes[0],"mezcla");

//	strcpy(pthread_data->pasos_param[5].accion, "hornear");
//	strcpy(pthread_data->pasos_param[5].ingredientes[0],"pan");
	
//	strcpy(pthread_data->pasos_param[6].accion, "cortarLTCP");
//	strcpy(pthread_data->pasos_param[6].ingredientes[0],"lechuga");
//	strcpy(pthread_data->pasos_param[6].ingredientes[1],"tomate");
//	strcpy(pthread_data->pasos_param[6].ingredientes[2],"cebolla");
//	strcpy(pthread_data->pasos_param[6].ingredientes[3],"pepino");

//	strcpy(pthread_data->pasos_param[7].accion,"GANADOR");
//	strcpy(pthread_data->pasos_param[7].ingredientes[0],"pan");
//	strcpy(pthread_data->pasos_param[7].ingredientes[1],"mezcla");
//	strcpy(pthread_data->pasos_param[7].ingredientes[2],"lechuga");
//	strcpy(pthread_data->pasos_param[7].ingredientes[3],"tomate");
//	strcpy(pthread_data->pasos_param[7].ingredientes[4],"cebolla");
//	strcpy(pthread_data->pasos_param[7].ingredientes[5],"pepino");
	//inicializo los semaforos

	sem_init(&(pthread_data->semaforos_param.sem_mezclar),0,0);
	sem_init(&(pthread_data->semaforos_param.sem_salar),0,0);
	sem_init(&(pthread_data->semaforos_param.sem_empanar),0,0);
	sem_init(&(pthread_data->semaforos_param.sem_sarten),0,0);
	sem_init(&(pthread_data->semaforos_param.sem_cocinarT),0,0);
	sem_init(&(pthread_data->semaforos_param.sem_hornearT),0,0);
	sem_init(&(pthread_data->semaforos_param.sem_cortarLTCPT),0,0);
	//sem_init(&(pthread_data->semaforos_param.sem_sandwichListo),0,0);
	//inicializar demas semaforos aqui


	//creo los hilos a todos les paso el struct creado (el mismo a todos los hilos) ya que todos comparten los semaforos 
    int rc;
    int rd;
    int re;
    int rf;
    int rg;
    int rh;
    int ri;
    int rj;
    rc = pthread_create(&p1,                           //identificador unico
                            NULL,                          //atributos del thread
                                cortar,             //funcion a ejecutar
                                pthread_data);                     //parametros de la funcion a ejecutar, pasado por referencia
	//crear demas hilos aqui
    rd = pthread_create(&p1,NULL,mezclar,pthread_data);
    re = pthread_create(&p1,NULL,salar,pthread_data);
    rf = pthread_create(&p1,NULL,empanarMezcla,pthread_data);
    rg = pthread_create(&p1,NULL,cocinar,pthread_data);
    rh = pthread_create(&p1,NULL,hornear,pthread_data);
    ri = pthread_create(&p1,NULL,cortarLTCP,pthread_data);
    rj = pthread_create(&p1,NULL,armarSandwich,pthread_data);
	
	
	//join de todos los hilos
	pthread_join (p1,NULL);
	pthread_join (p1,NULL);
	pthread_join (p1,NULL);
	pthread_join (p1,NULL);
	pthread_join (p1,NULL);
	pthread_join (p1,NULL);
	pthread_join (p1,NULL);
	pthread_join (p1,NULL);
	//crear join de demas hilos


	//valido que el hilo se alla creado bien 
    if (rc){
       printf("Error:unable to create thread, %d \n", rc);
       exit(-1);
     }
   if (rd){
	printf("Error:unable to create thread, %d \n", rd);
	exit(-1);
     }
   if(re){
	printf("Error:unable to create thread, %d \n", re);
	exit(-1);
	}
   if(rf){
	printf("Error:untable to create thread, %d \n",rf);
	exit(-1);
	}
   if(rg){
	printf("Error:unable to create thread, %d \n", rg);
	exit(-1);
	}
   if(rh){
	printf("Error:unable to create thread, %d \n", rh);
	exit(-1);
	}
   if(ri){
	printf("Error:unable to create thread, %d \n", ri);
	exit(-1);
	}
   if(rj){
	printf("Error:unable to create thread, %d \n", rj);
	exit(-1);
	}

	// fclose(ak);
	//destruccion de los semaforos 
	sem_destroy(&sem_mezclar);
	sem_destroy(&sem_salar);
	sem_destroy(&sem_empanar);
	sem_destroy(&sem_sarten);
	sem_destroy(&sem_cocinarT);
	sem_destroy(&sem_hornearT);
	sem_destroy(&sem_cortarLTCPT);
	//sem_destroy(&sem_sandwichListo);
	//destruir demas semaforos 
	
	//salida del hilo
	 pthread_exit(NULL);
}


int main ()
{      // FILE *ak;
	//ak = fopen("Salida.txt","a");
	//creo los nombres de los equipos
	//printf("El ganador es el equipo:",Equipo); 
	int rc;
	//int rd;//,re,rf,rg,rh,ri,rj;
	int *equipoNombre1 =malloc(sizeof(*equipoNombre1));
	int *equipoNombre2 =malloc(sizeof(*equipoNombre2));
	int *equipoNombre3 =malloc(sizeof(*equipoNombre3));
	int *equipoNombre4 =malloc(sizeof(*equipoNombre4));


	*equipoNombre1 = 1;
	*equipoNombre2 = 2;
	*equipoNombre3 = 3;
	*equipoNombre4 = 4;

	//creo las variables los hilos de los equipos
	pthread_t equipo1; 
	pthread_t equipo2;
	pthread_t equipo3;
	pthread_t equipo4;
 
	//inicializo los hilos de los equipos
    rc = pthread_create(&equipo1,                           //identificador unico
                            NULL,                          //atributos del thread
                                ejecutarReceta,             //funcion a ejecutar
                                equipoNombre1);
    

    rc = pthread_create(&equipo2,                           //identificador unico
                            NULL,                          //atributos del thread
                                ejecutarReceta,             //funcion a ejecutar
                                equipoNombre2);
    

    rc = pthread_create(&equipo3,                           //identificador unico
                            NULL,                          //atributos del thread
                                ejecutarReceta,             //funcion a ejecutar
                                equipoNombre3);

    rc = pthread_create(&equipo4,                           //identificador unico
                            NULL,                          //atributos del thread
                                ejecutarReceta,             //funcion a ejecutar
                                equipoNombre4);
    //printf("El ganador es:%d\n",Equipo);

   if (rc){
       printf("Error:unable to create thread, %d \n", rc);
       exit(-1);
     } 
   

	//join de todos los hilos
	pthread_join (equipo1,NULL);
	pthread_join (equipo2,NULL);
	pthread_join (equipo3,NULL);
	pthread_join (equipo4,NULL);

	//printf("El ganador es el equipo:",Equipo);
    pthread_exit(NULL);
    fclose(ak);
    
    
}


//Para compilar:   gcc subway.c -o ejecutable -lpthread
//Para ejecutar:   ./ejecutable




