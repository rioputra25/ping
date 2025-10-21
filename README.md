# ping
real-time monitor
#!/usr/bin/env python3
import subprocess
import re
import time
import os

def check_ping(host="8.8.8.8", count=2):
    """Lakukan ping dan ambil data packet loss & latency."""
    try:
        result = subprocess.run(
            ["ping", "-c", str(count), host],
            capture_output=True, text=True
        )
        output = result.stdout

        # Ambil packet loss
        loss_match = re.search(r'(\d+)% packet loss', output)
        packet_loss = int(loss_match.group(1)) if loss_match else 100

        # Ambil rata-rata waktu (rtt)
        time_match = re.search(r'rtt min/avg/max/mdev = [\d\.]+/([\d\.]+)/', output)
        avg_latency = float(time_match.group(1)) if time_match else None

        return packet_loss, avg_latency
    except Exception as e:
        print(f"Error: {e}")
        return None, None

def analyze_quality(packet_loss, avg_latency):
    """Analisis kualitas jaringan."""
    if packet_loss is None or avg_latency is None:
        return "❌ Tidak dapat menganalisis jaringan."

    if packet_loss == 100:
        return "⚫ Jaringan tidak tersambung (100% packet loss)."

    if avg_latency < 50 and packet_loss == 0:
        return f"🟢 Sangat Baik"
    elif avg_latency < 100 and packet_loss <= 5:
        return f"🟡 Cukup"
    elif avg_latency < 200 and packet_loss <= 20:
        return f"🔴 Buruk"
    else:
        return f"⚫ Sangat Buruk"

def clear_screen():
    os.system("clear" if os.name == "posix" else "cls")

def main():
    clear_screen()
    print("=== Real-time Network Quality Monitor ===")
    host = input("Masukkan alamat yang ingin diping (contoh: google.com): ").strip()
    if not host:
        host = "8.8.8.8"

    print(f"\nMemantau jaringan ke {host} setiap 5 detik...\nTekan Ctrl + C untuk berhenti.\n")

    try:
        while True:
            packet_loss, avg_latency = check_ping(host)
            quality = analyze_quality(packet_loss, avg_latency)

            clear_screen()
            print("=== Real-time Network Quality Monitor ===\n")
            print(f"Target Host     : {host}")
            print(f"Packet Loss     : {packet_loss}%")
            if avg_latency:
                print(f"Rata-rata Delay : {avg_latency:.2f} ms")
            print(f"Kualitas        : {quality}")
            print("\nTekan Ctrl + C untuk keluar...")
            time.sleep(5)

    except KeyboardInterrupt:
        print("\nMonitoring dihentikan oleh pengguna. 👋")

if __name__ == "__main__":
    main()
