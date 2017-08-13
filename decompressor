/**
 * A Program to decompress unheaded files compressed with a 12 bit - fixed width LZW algorithm
 * 
 * Usage: decompressor inputfile [outputfile]
 * 
 * Written by Laura Barlow
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]){
    
    //check for correct usage
    if(argc != 2 && argc != 3){
        fprintf(stderr, "Usage: decompressor inputfile [outputfile]\n");
        return 1;
    }
    
    //check input file can be opened
    FILE *compressed = fopen(argv[1], "r");
    if(compressed == NULL){
        fprintf(stderr, "input file could not be opened\n");
        return 2;
    }
    
    //check output file can be opened
    FILE *uncompressed = fopen((argc == 3) ? argv[2] : "decompressed.txt", "w");
    if(uncompressed == NULL){
        fprintf(stderr, "output file could not be opened\n");
        fclose(compressed);
        return 3;
    }    
        
    //finding length of input file
    fseek(compressed, 0, SEEK_END);
    long length = ftell(compressed);
    fseek(compressed, 0, SEEK_SET);
    
    //determining if there is padding at the end 
    int sixteenbits=0;
    if(length%3 != 0){
        length = length-2;
        sixteenbits=1;
    }
    
    int read1, read2, value1, value2=0;
    char *string1;
    char string2[2];
    char *dictionary[3840];
    int dictionaryCounter=0;
    long range = length*2/3;
    unsigned int a, b, c;
    
    //iterating across input file
    for(long i = 0; i < range; ++i){
        value1 = value2;
        
        //read in new bytes on the first pass and when i is odd
        if((i == 0 || i%2 == 1) && i != range-1){
            a = fgetc(compressed);
            b = fgetc(compressed);
            c = fgetc(compressed);
            read1 = (a<<4) + (b>>4);
            read2 = ((b&15)<<8) + c;
            if(i == 0){
                value1 = read1;
                value2 = read2; 
            }else{
                value2 = read1;
            }
        }else{
            value2=read2;
        }
        
        //strings 1 and 2 made using either ASCII codes or dictionary entries
        if(value1 < 256){
            string1=(char*)malloc(2);
            if(string1==NULL){
                fprintf(stderr, "can't allocate memory\n");
                return 4;
            }
            string1[0] = value1;
            string1[1] = '\0';
        }else{
            string1 = malloc(strlen(dictionary[value1-256])+1);
            strcpy(string1, dictionary[value1-256]);
        }
        if(value2<256){
            string2[0] = value2;
        }else if(value2-256==dictionaryCounter){
            string2[0] = string1[0];
        }else{
            string2[0] = dictionary[value2-256][0];
        }
        string2[1] = '\0';
        
        //adding string to output file
        fprintf(uncompressed, "%s", string1);
        
        //adding string1 and the next character to dictionary
        dictionary[dictionaryCounter] = realloc(string1, strlen(string1)+2);
        if(dictionary[dictionaryCounter]==NULL){
            fprintf(stderr, "having trouble reallocating memory\n");
            return 5;
        }
        strcat(dictionary[dictionaryCounter], &string2[0]);
        ++dictionaryCounter; 
        
        //resetting to initial dictionary
        if(dictionaryCounter>3839){
            for(int n=0; n<3840; ++n){
                free(dictionary[n]);
            }
            dictionaryCounter=0;
        }
    }
    
    //dealing with odd 16 bit end piece
    if(sixteenbits==1){
        a = fgetc(compressed);
        b = fgetc(compressed);
        read1 = ((a & 15)<<8) + b;
        if(read1<256){
            string1=(char*)malloc(2);
            string1[0] = read1;
            string1[1] = '\0';
        }else{
            string1 = dictionary[read1-256];
        }
        fprintf(uncompressed, "%s", string1);
    }
    
    //finishing up
    for(int n=0; n<dictionaryCounter; ++n){
        free(dictionary[n]);
    }
    fclose(compressed);
    fclose(uncompressed);
    return 0;
}
