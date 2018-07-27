# sample tcp server

#include <stdio.h><br>
#include <string.h>   //strlen<br>
#include <stdlib.h><br>
#include <errno.h><br>
#include <unistd.h>   //close<br>
#include <arpa/inet.h>    //close<br>
#include <sys/types.h><br>
#include <sys/socket.h><br>
#include <netinet/in.h><br>
#include <sys/ioctl.h><br>
#include <sys/time.h> //FD_SET, FD_ISSET, FD_ZERO macros<br>
  
#define TRUE   1<br>
#define FALSE  0<br>
#define PORT 9999<br>
 

int main(void)
{
	int sock, activity, new_sock, flags;
	struct sockaddr_in server_addr, client_addr;
	char buf[512];
	fd_set readfs;
	socklen_t client_addr_size;
	char msg[] = "Data Received !\n";
	int opt = 1;
	int recvBytes;

	struct timeval timeout;
	timeout.tv_sec = 20;
	timeout.tv_usec = 0;	

	sock = socket(AF_INET, SOCK_STREAM,0);
	//setsockopt(master_socket, SOL_SOCKET, SO_REUSEADDR, (char *)&opt, sizeof(opt))
	server_addr.sin_family = AF_INET;
	server_addr.sin_addr.s_addr = INADDR_ANY;
	server_addr.sin_port = htons(PORT);

	bind(sock,(struct sockaddr *)&server_addr,sizeof(server_addr));
	listen(sock, 3);
	printf("Waitiing for connections\n");

	client_addr_size = sizeof(client_addr);
	new_sock = accept(sock, (struct sockaddr *)&client_addr, &client_addr_size);
	printf("Client address: %s\n", inet_ntoa(client_addr.sin_addr));

	//flags = fcntl(new_sock,F_GETFL,0);
	//assert(flags != -1);
 	//fcntl(new_sock, F_SETFL, flags | O_NONBLOCK);
	//ioctl(new_sock, FIONBIO, &opt);

	while(TRUE)
	{
		//printf("sdsdsd\n");

		FD_ZERO(&readfs);
		FD_SET(new_sock, &readfs);

		activity = select(new_sock + 1, &readfs, NULL, NULL, &timeout);

		if(activity <= 0){ continue; }
		
		if (FD_ISSET(new_sock, &readfs))
		{
			FD_CLR(new_sock, &readfs);
			memset(&buf,0xFF,512);
			recvBytes = recv(new_sock, buf, sizeof(buf), 0);
			printf("Data: %s\n", buf);
			send(new_sock, &msg, sizeof(msg), 0);
			printf("Data Sent!\n");
		}
	}
	close(sock);
}
