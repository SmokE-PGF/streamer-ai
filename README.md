# Streamer-AI
Python based AI that uses Deep Neural Networks, Neuroevolution and Streamlabs APIs to live stream games while commentating over them at the same time.

## Running The program
Usage:
```bash
python3 main.py --gen [generations] --file [file_name] \
    --config [config]
```
This writes to a file called `say.txt`.
#### --gen
Specify the number of generations to run the evolution for
#### --file
File to load the checkpoint from. `checkpoints/` has saved checkpoints for generation
`2492` and `2284`.
#### --config
Configuration file for NEAT. `neat.config` contains the one used to train.

## Requirements
Install the requirements with
```bash
sudo make
```

#### Alternative
Install the python requirements with
```bash
pip install -r requirements.txt
```
and install `mpg123`, `fceux` and add them to `PATH`.

## Getting the API Key
* Go to `streamlabs.com` then sign in using your account. Then go to API settings, 
create a new app and copy the client ID and client Secret. Click the `Sample Authentication URL` below and then copy
the code in the URL.

* Call a `POST` request to `/token`. Make sure to use the same Redirect URI as set up in the app. Example:
```bash
curl --request POST \
     --url 'https://streamlabs.com/api/v1.0/token' \
     -d 'grant_type=grant_type&client_id=client_id&client_secret=client_secret&redirect_uri=redirect_uri'
 ```
 Example return:
 ```bash
 {
  access_token: 'loXk8FTOFwKfrLP3bGCnJldBxuGX03a03iQdxR8A',
  token_type: 'Bearer',
  refresh_token: 'IXCGDha46Q4eHBKrijmAqUwScbsMSuBy9IopXp80'
}
 ```
 
 * Authorize this `access_token` for the scope `socket.token` using `/authorize`. Example:
 ```bash
 curl --request GET \
  --url 'https://streamlabs.com/api/v1.0/authorize?response_type=response_type&client_id=client_id&redirect_uri=redirect_uri&scope=socket.token'
  ```
 * Use the `access_token` to get a `socket_token`. Call a `GET` request to `/socket/token`. Example:
```bash
 curl --request GET \
  --url 'https://streamlabs.com/api/v1.0/socket/token?access_token=access_token'
```
You can also use python's `requests` library instead of `curl`
* Install with `pip install requests`
* Example to get `socket_token`:
```python
import requests

url = "https://streamlabs.com/api/v1.0/socket/token"

querystring = {"access_token":"access_token"}

response = requests.request("GET", url, params=querystring)

print(response.text)
```
Put the `socket_token` in a file called `config`, with the following format, shown in
`config.example`:
```
socket_token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
## How it works
For an explanation for how the game runs, watch [this video](https://www.youtube.com/watch?v=hNDkjy2rXG4&).
### Short Explanation
#### How the game is played
This program uses a mathematical model, called a neural network, which simulates the brain of a human being. 
A neural network works by taking inputs and outputting probabilities for each of the outputs. This can be accomplished
by using a sigmoid function. <br><br>
![Sigmoid](https://qph.fs.quoracdn.net/main-qimg-07066668c05a556f1ff25040414a32b7)
<br><br>
`Neuroevolution of Augmenting Topologies`, or `NEAT` is what this project uses. The way standard
`neuroevolution` works is by randomly initializing a population of neural networks and
using survival of the fittest to get the best model. The best networks in each generations
are bred and some mutations are introduced. `NEAT` introduces features like speciation to
make a much more effective neuroevolution model. Neuroevolution is known to do better than standard
reinforcement learning models.<br>
#### How the commentary works
Using a `socketio` client for python, we can establish a connection to `Streamlab's Socket API`.
This API returns every alert in JSON format. We can decode these JSONs to return statements
thanking the donator, subscriber, member, superchat donator, etc. <br>Example:
```python
import socketio

URL = "http://example.com/socket-api" # Change this to whatever Socket API you are using
sio = socketio.Client()

@sio.on('connect')
def connect():
    # This function is called when the connection is established
    print("Connected")
    
# The event in quotes depends on your API, check the documentation
@sio.on("event")
def event(data):
    print(data)

# Connect to the URL specified above
sio.connect(URL)
```
The rest of the audio uses random number generations to generate the sentences. A sentence is picked from a list of sentences, 
stored in `get_sentences.py`. <br>
#### Playing the audio
The audio is converted to speech using `gtts`, Google's text-to-speech converter.
This is far more superior than `pyttsx`, which bugs out quite a bit.<br>
The audio is then played using `mpg123`, called by `os.system`.

## Future Work
* Integrate the streaming directly into the program, preventing the need for OBS.
* Prevent the game reopening after each genome.
* Improve commentary in general.
