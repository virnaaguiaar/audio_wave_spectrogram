# Converter um áudio para o formato de onda e de espectrograma 

### Reconhecimento de Voz

FUNÇÃO PARA GRAVAR:

    def record_audio(filename, duration=5):
      FORMAT = pyaudio.paInt16  
      CHANNELS = 1  # (1-mono, 2-estéreo)
      RATE = 44100 
      CHUNK = 1024  # frames/bloco
  
      audio = pyaudio.PyAudio()
  
      stream = audio.open(format=FORMAT,
                          channels=CHANNELS,
                          rate=RATE,
                          input=True,
                          frames_per_buffer=CHUNK)
  
      print("Fale: ...")
      frames = []
      
      for i in range(0, int(RATE / CHUNK * duration)):
          data = stream.read(CHUNK)
          frames.append(data)
  
      print("Gravação concluída.")
  
      stream.stop_stream()
      stream.close()
      audio.terminate()
  
      # Salva em um arquivo WAV
      wf = wave.open(filename, 'wb')
      wf.setnchannels(CHANNELS)
      wf.setsampwidth(audio.get_sample_size(FORMAT))
      wf.setframerate(RATE)
      wf.writeframes(b''.join(frames))
      wf.close()

FUNÇÃO PARA RECONHECER:

    def recognize_speech(audio_file):
      recognizer = sr.Recognizer()
      with sr.AudioFile(audio_file) as source:
          audio_data = recognizer.record(source)
  
      try:
          text = recognizer.recognize_google(audio_data, language='pt-BR')
          print("Texto reconhecido:", text)
      except sr.UnknownValueError:
          print("Não foi possível entender a fala.")
      except sr.RequestError as e:
          print("Erro ao solicitar reconhecimento de fala:", e)

![Captura de tela 2024-05-03 172000](https://github.com/virnaaguiaar/audio_wave_spectrogram/assets/85238057/89114712-b00a-4392-935c-566fd7275d69)

FUNÇÃO PARA 'PLOTAR' O ÁUDIO:


    def display_audio(filename):
     display(Audio(filename))

![Captura de tela 2024-05-03 172013](https://github.com/virnaaguiaar/audio_wave_spectrogram/assets/85238057/381c2cdf-7e95-48fb-9644-3188df3d4440)



CHAMADOS:

    record_audio(audio_filename, duration=5)
    recognize_speech(audio_filename)
    display_audio(audio_filename)

## Onda
FUNÇÃO PARA PLOTAR ÁUDIO EM ONDA:

    def plot_waveform(filename):
        with wave.open(filename, 'rb') as wf:
            num_channels = wf.getnchannels()
            sample_width = wf.getsampwidth()
            frame_rate = wf.getframerate()
            num_frames = wf.getnframes()
    
            raw_data = wf.readframes(num_frames)
    
        if sample_width == 1:
            data = np.frombuffer(raw_data, dtype=np.uint8)
            data = (data - 128) / 128.0  # Normaliza os valores para o intervalo [-1, 1]
        elif sample_width == 2:
            data = np.frombuffer(raw_data, dtype=np.int16)
            data = data / (2 ** 15)
        else:
            raise ValueError("Unsupported sample width")
    
        time = np.linspace(0, num_frames / frame_rate, num_frames)
    
        # Plota o áudio em formato de onda
        plt.figure(figsize=(11, 6))
        plt.plot(time, data, color='b')
        plt.xlabel('Tempo (s)')
        plt.ylabel('Amplitude')
        plt.title('Forma de onda do áudio')
        plt.grid(True)
        plt.show()

    plot_waveform(audio_filename) #chamada da função


![Captura de tela 2024-05-03 172027](https://github.com/virnaaguiaar/audio_wave_spectrogram/assets/85238057/52b8311f-921b-4813-9fc3-7d37f3432e79)

## Espectrograma

FUNÇÃO PARA PLOTAR O ESPECTROGRAMA:

    def plot_spectrogram(filename):
        y, sr = librosa.load(filename, sr=44100) #sr = sample rate (taxa de amostragem)
    
        S = librosa.feature.melspectrogram(y=y, sr=sr) #calcula o spectro com o STFT do librosa
    
        S_dB = librosa.power_to_db(S, ref=np.max) #conversão para db
    
        # Plota o espectrograma
        plt.figure(figsize=(11, 6))
        librosa.display.specshow(S_dB, sr=sr, x_axis='time', y_axis='mel')
        plt.title('Espectrograma do Áudio')
        plt.xlabel('Tempo (s)')
        plt.ylabel('Frequência (Hz)')
        plt.tight_layout()
        plt.show()

    plot_spectrogram(audio_filename) #chamada da função


![Captura de tela 2024-05-03 172038](https://github.com/virnaaguiaar/audio_wave_spectrogram/assets/85238057/678e14e2-c81c-4f18-8c1e-cfc9fe56d771)


    audio_filename = "audio.wav"

