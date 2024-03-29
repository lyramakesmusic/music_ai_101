## Sample Generation

- **Dance Diffusion** - Train an unconditional diffusion model on your sounds to get similar ones. It generates 1-2 seconds (usually 65536 or 131072 audio samples) of whatever sample rate it was trained on (usually 44.1 or 48khz). It's especially good for things like neurobass, which typically has a lot of noise that masks the noisy imperfections the diffusion process leaves. i have successfully trained it on kicks, snares, hardstyle kicks, top loops, neuro basses, and pads. 
- **Musicgen** - Either use pretrained for things like percussion loops, OR finetune it on 30-second chunks of songs. This model makes longer loops and song snippets, not so good for sample gen. You'll probably want to use Dance Diffusion for that. This model is limited to 32khz and mono (the stereo version is... not particularly good. Way too wide/imbalanced) 
- **Stable Audio** - As of Mar. 2024, there is only the web interface, no model you can run yourself. However, there is plans to release the model weights in the near future, and we'll be able to fairly easily finetune it on your sounds like the other models. Unlike musicgen, this model does 44.1khz stereo - much better for sampling. 
- **Other Generative Models** - Tools like Riffusion, Suno, Soundry, etc are also potentially worth playing with, although they all have limitations. Riffusion is usually pretty terrible quality since it generates image spectrograms with a finetuned stable diffusion and reverses those to audio instead of directly denoising audio samples or VAE latents. Suno does "full song generation" but focuses more on lyrics and getting coherent vocals, and the beats are not really sampling quality. Soundry is a decent service if you want to generate dubstep sounds, but there's no way to finetune or generate locally with it.

## Other Tools

- **Demucs** is a stem splitter. Given a mixed input track, it tries to separate it into 4 stems (Drums, Bass, Vocals, Other) that mix together into the original track. It does leave pretty bad compression artifacts, but works well enough to use for bootleg remixes. FL Studio 21 has it built in as [Stem Separator](https://www.image-line.com/fl-studio-learning/fl-studio-online-manual/html/playlist.htm#audio_clip_extractstems), and there's a variety of options if you use a different DAW. There are websites you can pay for that do this, like lalal.ai and similar, but there's no need if you have a decent GPU- instead you can download **[Ultimate Vocal Remover aka UVR5](https://ultimatevocalremover.com/)** which is 100% free and has a wide variety of splitting options. Not only does it have Demucs (change Process Method to `Demucs` and Demucs Model to `v4 | htdemucs`), it also has DeReverb models that will try and remove reverb/echo/ambiance from a track. You get these by clicking `Download More Models` and selecting an arch/model from the radio/dropdowns. A couple of the DeReverb models are `MDX-Net Model: Reverb HQ by FoxJoy` and `UVR-DeEcho-DeReverb`. These models are crazy fun, and I'd recommend trying a variety of them to get familiar.
- **RVC** is a voice transfer model. This is how everyone is making those Minecraft Villager remixes, or Squidward singing, or those Presidents saying dumb stuff Youtube shorts. You can download (and use!!) models for specific voices **[from this site (voice-models.com)](https://voice-models.com/)**. If you click the `Run` button next to a voice and sign in to the site that appears, you can run them without code or fancy setup, but you can also run it in Colab or similar. There are a wide variety of notebooks available for that, googling "RVC inference colab" gives you a bunch of options. I ran into environment errors installing `RVC-webui` locally, weird dependency conflicts that I was unable to easily resolve. If you're just looking to run it a couple times, just use the site above- it's much easier than dealing with uploading checkpoints to colab or solving weird code bugs. 

### What do I do with these models?

Literally just sample from them. Make some snares with Dance Diffusion. Paulstretch some Stable Audio outputs into a weird pad. Tune Musicgen on top loops. Send VR Bass loops through RVC Squidward and distort them. Generate some DnB with Google's [MusicFX](https://aitestkitchen.withgoogle.com/tools/music-fx) and replace every single sound with your own. Make a Mariah Carey riddim remix with Demucs stems. You're a producer, you know what to do, just have fun!

## Training Recipes

### Training (and running) Dance Diffusion

[Finetuning notebook](https://colab.research.google.com/github/Harmonai-org/sample-generator/blob/main/Finetune_Dance_Diffusion.ipynb), [Inference notebook](https://colab.research.google.com/github/Harmonai-org/sample-generator/blob/main/Dance_Diffusion.ipynb)

**Ingredients**:
- At least 100 audio samples (preferably closer to 1000)
- The Finetuning notebook linked above
- A Google account (you'll need Drive and Colab)

**Steps**:
- Upload all your samples to a folder in Google Drive

![Screenshot showing some Au5 bass loops in a Google Drive folder](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/dd_dataset_in_drive.png)

- In the finetuning notebook, change `TRAINING_DIR` to the path to that folder, making sure it matches the format `/content/drive/MyDrive/your_folder_here`

![Screenshot showing a different training dir in the notebook](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/dd_data_path.png)

- Adjust `SAMPLE_RATE` and `SAMPLE_SIZE` - you should match the sample rate of your training audio. The default `SAMPLE_SIZE` gives ~1.5 seconds of audio, doubling it to 131072 gives you ~2.5-3 seconds at the cost of more training & inference time. You can also change the run name if you like.

![Screenshot showing altered sample rate and length](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/dd_sample_info.png)

- You should probably make a [wandb](https://wandb.ai/) account, it will let you track the model progress as it trains and displays generated samples during training. The numbers will often be all over the place, so ear test (listening to the demos) is your best indicator of actual progress. The notebook should guide you through what to do with the account, and you can always skip this also (but the demos are really nice!).

![Screenshot of wandb asking for authorization](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/dd_wandb_login.png)
  
- Press all the run buttons in the notebook from top down, starting with `Check GPU Status`. Follow any additional prompts it gives you as it runs, but it should be mostly hands-free after the last cell has started. Now.... you wait. It takes a few hours to train, typically. My longest training run lasted 3 days, and my shortest was 1.5 hours. If everything worked correctly, colab should output this:

![Screenshot of Dance Diffusion successfully running](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/dd_running.png)

- If you click the wandb link it gave you, you'll hear the first batch of demos- they're generated before it starts training on your sound, so be patient and wait for the next demos 250 steps later.

![Screenshot of wandb showing a demo audio](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/dd_wandb_demo.png)

- After every 500 steps, a copy of the trained model will be saved to your Google Drive, in `/content/drive/MyDrive/AI/models/DanceDiffusion/finetune` or whatever you might possibly have changed that line to. You'll be able to download this and run it locally (after writing some inference code and installing pytorch with gpu etc), or point the inference notebook to that checkpoint.

![Screenshot of Google Drive showing a trained ckpt](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/dd_ckpt_in_drive.png)

- Inference: You'll need to run the [Inference notebook](https://colab.research.google.com/github/Harmonai-org/sample-generator/blob/main/Dance_Diffusion.ipynb) also linked above. Scroll down to the "Create the Model" section, change the model type to "Custom" and link the trained checkpoint using the same `/content/Drive/MyDrive/` format. You'll need to remember what sample rate and length you trained it on, because the inference notebook defaults to 65536@16k- your model should be in the ballpark of 65536@44100 or 131072@48k. Go ahead and put those values in.

  ![Screenshot of the Dance Diffusion inference notebook showing the model selection dropdown set to Custom](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/dd_infer_custom_model.png)

Run all the cells, again starting at the top, and going down to `Generate new sounds`. The ones below that are for different types of inference, ie audio2audio, we don't need those to run. I do recommend playing with them, they can make some cool stuff, but not needed just to get sounds. Enjoy your samples! If you want to use some pretrained models before trying to finetune your own, here are a few I trained: 

| Model      | Sample Length | Sample Rate | Finetuning Steps | Link |
| ----------- | ----------- | ----------- | ----------- |----------- |
| Neuro & dubstep basses | 131072 | 44100 | 6500 steps | [Download](https://drive.google.com/file/d/1-IL74bhrZKrYbjfmEmG70dEWezFfxJgW/view?usp=sharing) (3.3GB)|
| Drum fills   | 131072 | 44100 | 1500 steps | [Download](https://drive.google.com/file/d/1-B2S1AhyDz4eMK9n3xRnPclW4OzV7lLf/view?usp=sharing) (3.3GB)        |
| Hardstyle kicks | 65536 | 44100 | 5500 steps | [Download](https://drive.google.com/file/d/1-S6gGj5qW1mKQP4yxNyO4XXpUzZkjYsU/view?usp=sharing) (3.3GB)  |
| Top loops | 131072 | 44100 | 3500 steps | [Download](https://drive.google.com/file/d/1-IMK47o1HhqCSHoolsEx-cKnIijkrDDC/view?usp=sharing) (3.3GB) |
| Snares | 65536 | 44100 | 0 steps | Download (3.3GB) |

Quality *will* vary! You'll need to do multiple generations and manually slice up the samples for good results.

### Training (and running) Musicgen

[Detailed explanation](https://colab.research.google.com/drive/13tbcC3A42KlaUZ21qvUXd25SFLu8WIvb), [I just want to run it](https://colab.research.google.com/drive/1VX8tMAfyWVEHZiyviuovUgKXq1GpKcdR)

Info on training this model is outside the scope of this readme, a detailed guide (screenshot below) is linked above. If you want to finetune without bothering with technical setup, you can use the second link to finetune with a youtube playlist link. If you want to just run it without finetuning, you can use the [Musicgen Huggingface space](https://huggingface.co/spaces/facebook/MusicGen). 

![Screenshot of the Musicgen Training colab, showing detailed explanations of dataset setup and training VRAM requirements](https://raw.githubusercontent.com/lyramakesmusic/music_ai_101/main/musicgen_notebook.png)

### Training (and running) Stable Audio

[Stable Audio Tools repo](https://github.com/Stability-AI/stable-audio-tools)

There's no Stable Audio model weights currently available, this section will be updated on model release. I could explain the tuning process already, but it won't be very useful without weights since training from scratch is ***very expensive***

## Info for Nerds

### How do these models even work??

Oh boy here we go. [TBA later, there's many details to figure out how to communicate]

#### Autoencoders

![Diagram showing architecture of an Autoencoder, with identical data on each side being compressed into a latent representation](https://miro.medium.com/v2/resize:fit:600/1*nqzWupxC60iAH2dYrFT78Q.png)

#### Diffusion

![GIF of RGB noise being gradually denoised into a satellite image of a house](https://aurorasolar.com/wp-content/uploads/2022/07/Roof-Deepfakes-64-steps.gif)
