/* table.c 
 * table: Hash Tables
 * ECE 223, Fall 2015
 * Huzefa Dossaji 
 * hdossaj
 *
 * PURPOSE: This is the modular design to implement hash tables. Functions like construct, destruct, insert, remove,
 * find are all implemented here.  
 *	
 * ASSUMPTION: User would like to create a hash table that is much faster than other ADTS
 *
 * BUGS: THERE ARE NO BUGS DETECTED EVEN WITH ALL MY EXTENSIVE TESTING
 *
 */

#include <stdlib.h>
#include <stdio.h>
#include <assert.h>
#include <math.h>
#include <string.h>
#include <ctype.h>
#include <time.h>
#include <unistd.h>

#include "table.h"

#define max(a,b) (a>b?a:b)
int TABLESIZE;
hashkey_t NOTFILLED = 0;
hashkey_t DEL = 1;

// creating the new hash table
table_t *table_construct(int table_size, int probe_type){
	TABLESIZE = table_size;
	table_t *table = NULL;
	int a;
	
	if(table_size < 1) return NULL;
	
	//Create the header table_t that points to the pointers to memory blocks
	if((table = malloc(sizeof(table_t))) == NULL){
		return NULL;
	}
	
	//Now actually create space for the pointers to the memory blocks
	if((table->oa = malloc(sizeof(table_entry_t) *  table_size)) == NULL){
		return NULL;
	}
	
	for(a = 0; a< table_size; a++){
		table->oa[a].key = NOTFILLED;
		table->oa[a].data_ptr = NULL;
	}
	
	table->table_size_M = table_size;
	table->probe=probe_type;
	table->entries = 0;
	table->recent_probes = 0;
	table->deleted_entries = 0;
	
	return table;

}

/* Sequentially remove each table entry (K, I) and insert into a new
 * empty table with size new_table_size.  Free the memory for the old table
 * and return the pointer to the new table.  The probe type should remain
 * the same.
 *
 * Do not rehash the table during an insert or delete function call.  Instead
 * use drivers to verify under what conditions rehashing is required, and
 * call the rehash function in the driver to show how the performance
 * can be improved.
 */
table_t *table_rehash(table_t * T, int new_table_size){
	table_t *new;
	new = table_construct(new_table_size, T->probe);
	int a;
	
	for(a=0;a<T->table_size_M;a++){
		if(T->oa[a].key != NOTFILLED && T->oa[a].key != DEL){
			table_insert(new, T->oa[a].key, T->oa[a].data_ptr);
			table_delete(T, T->oa[a].key);
		}
	}
	table_destruct(T);
	return new;
	

}

/* returns number of entries in the table */
int table_entries(table_t * table){
	return table->entries;
}

/* returns 1 if table is full and 0 if not full. */
int table_full(table_t * table){
	if(table->entries == (TABLESIZE -1)){
		return 1;
	}
	return 0;
}

/* returns the number of table entries marked as deleted */
int table_deletekeys(table_t * table){
	return table->deleted_entries;
}
   
/* Insert a new table entry (K, I) into the table provided the table is not
 * already full.  
 * Return:
 *      0 if (K, I) is inserted, 
 *      1 if an older (K, I) is already in the table (update with the new I), or 
 *     -1 if the (K, I) pair cannot be inserted.
 */
int table_insert(table_t * table, hashkey_t K, data_t I){
	int h = K % table->table_size_M; 
	int p = probe_function(table, K); 
	int probe=1;
	data_t *T=NULL;
	int location_del = -1;
	
	if(table_full(table) == 1 && table_retrieve(table,K) == NULL){
		return -1;
	}
	while(table->oa[h].key != NOTFILLED){

		if(table->oa[h].key == DEL && location_del == -1){
			location_del = h;
		}
		if(table->oa[h].key == K){
			T=table->oa[h].data_ptr;
			free(T);
			table->oa[h].data_ptr = I;
			table->recent_probes = probe;
			return 1;
		}
		if(table->probe == 2){
			p++;
		}
		h = h-p;
		if(h<0){
		   h=h+table->table_size_M;
		}
	
		if(probe == table->table_size_M){
			if(location_del != -1){
				table->oa[location_del].key = K;
				table->oa[location_del].data_ptr = I;
				table->entries++;
				table->deleted_entries--;
				table->recent_probes = probe;
				return 0;
			}
			table->recent_probes = probe;
			return -1;
		}
		probe++;
	}
	if(location_del != -1){
		table->oa[location_del].key = K;
		table->oa[location_del].data_ptr = I;
		table->entries++;
		table->deleted_entries--;
		table->recent_probes = probe;
		return 0; 
	}
	table->oa[h].key = K;
	table->oa[h].data_ptr = I;
	table->entries++;
	table->recent_probes = probe;
	return 0;
}

/* Delete the table entry (K, I) from the table.  
 * Return:
 *     pointer to I, or
 *     null if (K, I) is not found in the table.  
 *
 * See the note on page 490 in Standish's book about marking table entries for
 * deletions when using open addressing.
 */
data_t table_delete(table_t * table, hashkey_t K){
	int p = probe_function(table, K); 
	int h = K % table->table_size_M;
	int probe=1;
	data_t * data;
	

	while(table->oa[h].key != NOTFILLED ){
		if(table->oa[h].key == K){
			table->oa[h].key = DEL;
			data = table->oa[h].data_ptr;
			table->oa[h].data_ptr = NULL;
			table->recent_probes = probe;
			table->entries--;
			table->deleted_entries++;
			return data;
		}
		if(table->probe == 2){
			p++;
		}
		h = h - p;
		if(h<0){
		   h = h + table->table_size_M;
		}
		if(probe == table->table_size_M){
			table->recent_probes = probe;
			return NULL;
		}
		probe++;
	}
	
	table->recent_probes = probe;
	return NULL;
}

/* Given a key, K, retrieve the pointer to the information, I, from the table,
 * but do not remove (K, I) from the table.  Return NULL if the key is not
 * found.
 */
data_t table_retrieve(table_t * table, hashkey_t K){
	int p = probe_function(table, K); 
	int h = K % table->table_size_M;
	int probe=1;
	
	
	while(table->oa[h].key != NOTFILLED){
		if(table->oa[h].key == K){
			table->recent_probes = probe;
			return table->oa[h].data_ptr;
		}
		
		if(table->probe == 2){
			p++;
		}
		h = h - p;
		
		if(h<0){
		   h = h + table->table_size_M;
		}
		if(probe == table->table_size_M){
			table->recent_probes = probe;
			return NULL;
		}
		probe++;
	}
	
	table->recent_probes = probe; 
	
	return NULL;
}
/* Free all information in the table, the table itself, and any additional
 * headers or other supporting data structures.  
 */
void table_destruct(table_t * table){
	int a;
	//table_debug_print(table);
	for(a=0; a<table->table_size_M; a++){
		if(table->oa[a].key != NOTFILLED && table->oa[a].key != DEL){
			free(table->oa[a].data_ptr);
		}
	}
	free(table->oa);

	free(table);

}

/* The number of probes for the most recent call to table_retrieve,
 * table_insert, or table_delete 
 */
int table_stats(table_t *table){
	return table->recent_probes;
}

/* This function is for testing purposes only.  Given an index position into
 * the hash table return the value of the key if data is stored in this 
 * index position.  If the index position does not contain data, then the
 * return value must be zero.  Make the first
 * lines of this function 
 *       assert(0 <= index && index < table_size); 
 */
hashkey_t table_peek(table_t *table, int index){
	assert(0 <= index && index < table->table_size_M);
	
	if(table->oa[index].key == NOTFILLED || table->oa[index].key == DEL){
		return 0;
	}else{
		return table->oa[index].key;	
	}
	return 0;
}

/* Print the table position and keys in a easily readable and compact format.
 * Only useful when the table is small.
 */
void table_debug_print(table_t *table){
	int x;
	printf("\nSTART OF TABLE\n");
	for(x=0;x<table->table_size_M; x++){
		printf("Index: %d, Key: ", x);
		if(table->oa[x].key == DEL){
			printf("DELETED\n"); 
		}else if(table->oa[x].key == NOTFILLED)
		{
			printf("EMPTY\n");
		}else{
			printf("%d \n", table->oa[x].key);
		}
		
	}

}

// Function that calculates with probeing function to use. 
int probe_function(table_t * table, hashkey_t K){
	int size = table->table_size_M;
	if(table->probe == 0){
		return 1;
	}
	
	if(table->probe == 1){
		return max(1, (K/size) % size);	
	}
	
	if(table->probe == 2){
		return 0;
	}
	return 0;
}


