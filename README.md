import RPi.GPIO as GPIO
import MFRC522
import picamera
import Adafruit_DHT
import time

# Configuration des broches GPIO
LED_PIN = 17
BUZZER_PIN = 27
DHT_PIN = 4

# Configuration du lecteur RFID
lecteur = MFRC522.MFRC522()

# Configuration du capteur DHT11
capteur = Adafruit_DHT.DHT11

# Configuration des broches GPIO pour LED et Buzzer
GPIO.setmode(GPIO.BCM)
GPIO.setup(LED_PIN, GPIO.OUT)
GPIO.setup(BUZZER_PIN, GPIO.OUT)

# Fonction pour lire la carte RFID
def lire_carte_rfid():
    (status, tag_type) = lecteur.MFRC522_Request(lecteur.PICC_REQIDL)
    if status == lecteur.MI_OK:
        (status, uid) = lecteur.MFRC522_Anticoll()
        if status == lecteur.MI_OK:
            uid_str = ":".join([str(i) for i in uid])
            return uid_str
    return None

# Fonction pour lire la température et l'humidité
def lire_temperature_humidite():
    humidite, temperature = Adafruit_DHT.read_retry(capteur, DHT_PIN)
    return humidite, temperature

# Fonction pour autoriser l'accès
def autoriser_acces():
    print("Accès autorisé")
    GPIO.output(LED_PIN, GPIO.HIGH)
    GPIO.output(BUZZER_PIN, GPIO.HIGH)
    time.sleep(2)
    GPIO.output(LED_PIN, GPIO.LOW)
    GPIO.output(BUZZER_PIN, GPIO.LOW)

# Fonction pour refuser l'accès
def refuser_acces():
    print("Accès refusé")

# Gestion de la caméra
def prendre_photo():
    with picamera.PiCamera() as camera:
        camera.resolution = (1024, 768)
        camera.capture('photo.jpg')

# Boucle principale
try:
    print("En attente de carte RFID...")
    while True:
        carte_rfid = lire_carte_rfid()

        if carte_rfid is not None:
            print(f"Carte RFID détectée : {carte_rfid}")

            # Vérification des informations d'accès (personnaliser cette partie selon votre application)
            # Dans cet exemple, toute carte RFID est considérée comme un accès autorisé
            autoriser_acces()

            # Lecture de la température et de l'humidité
            humidite, temperature = lire_temperature_humidite()
            print(f"Température : {temperature}°C, Humidité : {humidite}%")

            # Prise d'une photo avec la caméra
            prendre_photo()

except KeyboardInterrupt:
    pass
finally:
    GPIO.cleanup()
