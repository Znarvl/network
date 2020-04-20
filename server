import socket, select
import sys
import queue
import threading
"""
PORT = 1234
IP = "127.0.0.1" #localhost

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
#SOL_SOCKET is the socket layer itself
#SO_REUSEADDR when busy when with other state return -1 (error)
server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)

server_socket.bind((IP, PORT))
server_socket.listen()

while True:
    client_socket, addr = server_socket.accept()
    print(f"Connection with {addr} is done!")
    data = ""
    while True:
        data = client_socket.recv(1024)
        print(data) # Outputs readable HTTP
        if not data: break
    client_socket.send(data)

    client_socket.close()

#CLIENT
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
    sock.connect((HOST, PORT))
    sock.sendall(b'Hello World')
    data = sock.recv(1024)

print('Received', repr(data))
#ENDCLIENT
"""

#headers = """GET http://zebroid.ida.liu.se/ HTTP/1.1
#            Host: http://zebroid.ida.liu.se/\r\n\r\n"""
HOST = "127.0.0.1"  # Proxy server IP (localhost)
PORT = 1234         # Proxy server port
max_conn = 5
buffer_size = 8192

# Sockets from which we expect to read
inputs = []

# Sockets to which we expect to write
outputs = []

# Outgoing message queues (socket:Queue)
message_queues = {}

def start():
    try:
        server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        server.bind((HOST, PORT))
        server.listen(max_conn)
        inputs.append(server)
        print("Server Started Successfully [%d]\n" % (PORT))
    except Exception as e:
        print ("Unable To Initialise Socket")
        sys.exit(2)

    while True:
        try:
            conn, addr = server.accept()
            data = conn.recv(buffer_size)
            header = get_header(data)
            webserver, port = conn_string(conn, header, addr) # Occasonally spits out NoneType object is not iterable
            with conn:
                proxy_server(webserver, port, conn, addr, data)
        except KeyboardInterrupt:
            server.close()
            print ("\n Proxy Server Closed by KeyboardInterrupt")
            sys.exit(1)
        except TypeError as msg:
            print(msg)
            pass
    server.close()


def get_header(data):
    header = data.split(b'\r\n\r\n')[0]
    header = header.decode("utf-8")
    return header


def get_contet(data):
    content = data.split(b'\r\n\r\n')[1]
    return content

def conn_string(conn, header, addr):
    try:
        first_line = header.split('\n')[0]
        url = first_line.split(' ')[1]
        http_pos = url.find("://")
        if (http_pos==-1):
            temp = url
        else:
            temp = url[(http_pos+3):]
        port_pos = temp.find(":")
        webserver_pos = temp.find("/")
        if webserver_pos == -1:
            webserver_pos = len(temp)
        webserver = ""
        port = -1
        if (port_pos == -1 or webserver_pos < port_pos):
            port = 80
            webserver = temp[:webserver_pos]
        else:
            port = int((temp[(port_pos+1):])[:webserver_pos-1])
            webserver = temp[:port_pos]

        print("webserver: ", webserver)
        print("port: ", port)
        return webserver, port
        #proxy_server(webserver, port, conn, addr, data)
    except Exception as e:
        pass

def proxy_server(webserver, port, conn, addr, data):
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((webserver, port))
        sock.send(data) # Initial GET Request

        while True:
            reply = sock.recv(buffer_size)
            #print("Reply: ", reply)
            if (len(reply) > 0):
                conn.send(reply)
                dar = float(len(reply))
                dar = float(dar / 1024)
                dar = "%.3s" % (str(dar))
                dar = "%s KB" % (dar)
                print("Request Done: %s => %s <=" %(str(addr[0]), str(dar)))
            else:
                break

        sock.close()
    except socket.error as msg:
        print("Socket Exception: ", msg)
        sock.close()
        conn.close()
        sys.exit(1)

start()
