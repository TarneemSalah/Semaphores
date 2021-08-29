# Semaphores
Familiarizing with concurrent programming and Handling races, synchronization and deadlock conditions.
In main function
1-declaration of threads
2-initialize semaphores
3-create threads
4-join threads
Code;
int main() {
pthread_t mcounter_threads[MaxThrad], mmonitor_thread,
mmcollector_thread;

int ids[MaxThrad],i;
sem_init(&S_counter, 0, 1);
sem_init(&S_of_buffer, 0, 1);
for ( i = 0; i < MaxThrad; i++) {
ids[i]=i+1;
pthread_create(&mcounter_threads[i], NULL, mcounter, &ids[i]);

}

pthread_create(&mmonitor_thread, NULL, mmonitor, NULL);
pthread_create(&mmcollector_thread, NULL, mmcollector, NULL);

pthread_join(mmcollector_thread, NULL);
pthread_join(mmonitor_thread, NULL);
for ( i = 0; i < MaxThrad; i++) pthread_join(mcounter_threads[i],
NULL);

return 0;
}

function of mcollector of threads

1-make integer to store the semaphore in
2-make temp integer to store the counter before reset it
3-make random sleeping
4-take the value of semaphore and check if there is thread
now using buffer
5-do semwait on buffer then sempost
Code:
void* mmcollector(void*u) {

int semaphore_val;

int temp_read_val;

sleep((rand() %(150 -50 + 1)) + 100);

sem_getvalue(&S_of_buffer, &semaphore_val);
if(semaphore_val < 1) printf("Collector thread: waiting to read
from buffer\n");
sem_wait(&S_of_buffer);
if(number_of_elements_in_buffer == 0) {
printf("Collector thread: nothing is in the buffer !\n");
}
else {
temp_read_val = buffer[buffer_read_pos];
printf("Collector thread: reading from buffer at position %d, read
value is %d\n",buffer_read_pos+1,temp_read_val);
if(buffer_read_pos == buffersize-1) buffer_read_pos = 0;
else buffer_read_pos++;
number_of_elements_in_buffer--;
}
sem_post(&S_of_buffer);

}

function of mmonitor of threads
1- make integer to store the semaphore in
2- make temp integer to store the counter before reset it
3- make random sleeping
4-take the value of semaphore and check if there is thread
5-do semwait and read counter then reset counter and then 6-take the
value of semaphore and check if there is thread now using buffer
7-do semwait and write tempcounter in buffer then sempost
8-make circular round in buffer
Code:
void* mmonitor(void*u) {
int semaphore_val;

int temp_counter_val;

Sleep((rand() %(200 -0 + 1)) + 0);

sem_getvalue(&S_counter, &semaphore_val);
if(semaphore_val < 1) printf("Monitor thread: waiting to read
counter\n");

sem_wait(&S_counter);
temp_counter_val = counter;
counter = 0;
printf("Monitor thread: reading a count value of
%d\n",temp_counter_val);
sem_post(&S_counter);

sem_getvalue(&S_of_buffer, &semaphore_val);
if(semaphore_val < 1) printf("Monitor thread: waiting to write to
buffer\n");

sem_wait(&S_of_buffer);
if(number_of_elements_in_buffer == buffersize) {
printf("Monitor thread: buffer full !\n");
}
else {
buffer[buffer_write_pos] = temp_counter_val;
printf("Monitor thread: writing to buffer at position
%d\n",buffer_write_pos+1);
///make circular round in buffer
