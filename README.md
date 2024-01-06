# Image-processing-drone
Contrôler un drone Tello par traitement d'images

# I- Contrôle drone par Python 

Vidéo projet : <https://www.tiktok.com/@__hakii__/video/7318342300592049440>
### a) Configuration drone
Une fois allumé, le drone Tello se transforme en point d'accès et génère son propre signal Wi-Fi. Il est alors possible, depuis notre PC, de se connecter à ce réseau. De cette manière, il va être possible de lui envoyer des requêtes pour lui faire effectuer des mouvements, mais également d'en recevoir, comme pour le flux vidéo de la caméra. Tous ces échanges se font par le protocole UDP.

Ce protocole est utilisé dans ce genre de situation car il propose une latence plus faible que d'autres protocoles, ce qui est crucial pour le contrôle en temps réel et le streaming vidéo.

Cependant, un des inconvénients est que ce protocole envoie simplement les paquets sans attendre de confirmation. Cela signifie qu'il est possible que certains paquets soient perdus en cours de route.

On importe la bibliotèque Tello :

```python
from  djitellopy import tello
```

On instancie un objet Tello et on se connecte au drone. On veille bien sûr à ce que notre ordinateur soit connecté au point d’accès du drone.

```python
drone=tello.Tello()
drone.connect()
```

### b) Commande controle temps réel

**`takeoff()`**: Fait décoller le drone.

**`land()`**: Fait atterrir le drone.
  
**`send_rc_control()`**

1. `left_right_velocity`: La vitesse de déplacement latéral gauche/droite.
2. `forward_backward_velocity`: La vitesse de déplacement avant/arrière.
3. `up_down_velocity`: La vitesse de montée/descente.
4. `yaw_velocity`: La vitesse de rotation autour de l'axe vertical (yaw).


## c) Programme contrôle clavier

On importe la bibliotèque keyboard. On effectue une boucle infinie qui vérifie si certaines touches du clavier sont pressées. On utilise la méthode **`send_rc_control()`** pour un contrôle en temps réel.

```python
from djitellopy import Tello
from time import sleep
import keyboard

drone = Tello()
drone.connect()

while True:
   
	key_mapping = {
	    "space": ("takeoff", drone.takeoff),
	    "esc": ("land", drone.land),
	    "f": ("Forward", lambda: drone.send_rc_control(0, 50, 0, 0)),
	    "b": ("Backward", lambda: drone.send_rc_control(0, -50, 0, 0)),
	    "r": ("Right", lambda: drone.send_rc_control(50, 0, 0, 0)),
	    "l": ("Left", lambda: drone.send_rc_control(-50, 0, 0, 0)),
	    "Up": ("Up", lambda: drone.send_rc_control(0, 0, 20, 0)),
	    "Down": ("Down", lambda: drone.send_rc_control(0, 0, -20, 0)),
	    "right": ("Rotate Right", lambda: drone.send_rc_control(0, 0, 0, 50)),
	    "left": ("Rotate Left", lambda: drone.send_rc_control(0, 0, 0, -50))
	}
        
     for key, (action, func) in key_mapping.items():
            if keyboard.is_pressed(key):
                print(action)
                func()

    drone.send_rc_control(0, 0, 0, 0) # On réinitialise à chaque fois la vitesse
    sleep(0.05)
```

Dans cette version, on utilise un dictionnaire dont la clé est la touche de notre clavier et la valeur est un tuple contenant le nom à afficher et la fonction à appeller.

- Toutes les fonctions sans paramètres peuvent directement être incluses dans le tuple.
- Les fonctions avec paramètres doivent être appelées par l’intermédiaire d’une fonction anonyme lambda. Lorsque `lambda: drone.send_rc_control(0, 50, 0, 0)` ****est appelée, elle execute ****`drone.send_rc_control(0, 50, 0, 0)`

Dans une boucle, on récupère toutes les mots clés possibles et on vérifie si on touche correspond à celui-ci est pressé.
