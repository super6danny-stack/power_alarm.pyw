import psutil
import time
import winsound
import threading
import sys

# Flag to kill the sound loop
stop_signal = False

def play_alarm_sound():
    global stop_signal
    while not stop_signal:
        # Check battery inside the sound loop for instant silence
        battery = psutil.sensors_battery()
        if battery and battery.power_plugged:
            break
            
        winsound.Beep(2500, 500)
        time.sleep(0.1)

def check_power():
    global stop_signal
    print("System active. Monitoring power status...")
    
    battery = psutil.sensors_battery()
    if battery is None:
        print("Error: No battery found.")
        return
        
    last_status = battery.power_plugged

    while True:
        battery = psutil.sensors_battery()
        current_status = battery.power_plugged

        # Case 1: Disconnected -> Start Alarm
        if last_status == True and current_status == False:
            print("ALERT: Power disconnected!")
            stop_signal = False
            # Start sound in a daemon thread
            alarm_thread = threading.Thread(target=play_alarm_sound, daemon=True)
            alarm_thread.start()
            
        # Case 2: Reconnected -> Stop Alarm Instantly
        elif current_status == True:
            if not stop_signal:
                print("Power restored. Silence.")
                stop_signal = True 

        last_status = current_status
        time.sleep(0.2) # Faster check for real-time response

if __name__ == "__main__":
    try:
        check_power()
    except KeyboardInterrupt:
        stop_signal = True
        print("\nExit.")
        sys.exit()
