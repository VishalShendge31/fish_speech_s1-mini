# fish_speech_s1-mini
Complete, up-to-date, terminal-based guide for running the Fish-Speech / OpenAudio-S1-Mini TTS pipeline on Ubuntu 22.04, using your Thorsten Emotional dataset. This workflow covers: environment, dependencies, model download, data prep, code extraction, semantic token generation, waveform synthesis, and audio playback/saving—with robust Linux tools

# Step 0: System Preparation
## Install system dependencies:
sudo apt update
sudo apt install python3 python3-pip python3-venv git portaudio19-dev libsox-dev ffmpeg alsa-utils sox

# Step 1: Create and Activate Python Environment
cd ~/fishspeech_german            # or your preferred workspace directory
python3 -m venv fish2
source fish2/bin/activate

# Step 2: Clone the Fish-Speech Repository
git clone https://github.com/fishaudio/fish-speech.git
cd fish-speech

# Step 3: Install Python Dependencies
pip install -e .
pip install huggingface_hub

# Step 4: Download the Model Checkpoints
huggingface-cli login   # Paste your HF token when prompted
huggingface-cli download fishaudio/openaudio-s1-mini --local-dir checkpoints/openaudio-s1-mini

# Step 5: Organize Your Dataset
fish-speech/
  ├── checkpoints/openaudio-s1-mini/
  ├── data/thorsten-emotional_v02/
      ├── amused/
      │   ├── *.wav
      ├── angry/
      │   ├── *.wav
      ├── ... (other emotions)
      └── thorsten-emotional-metadata.csv

Step 6: Extract Codes (.npy) from a WAV File
Run this from inside your fish-speech directory:
python3 fish_speech/models/dac/inference.py \
  -i data/thorsten-emotional_v02/amused/0aaf116f8774a140909231e0b610dc98.wav \
  -o codes_0.wav \
  --checkpoint-path checkpoints/openaudio-s1-mini/codec.pth \
  --device cuda

This creates:
codes_0.wav: reconstructed audio (not needed for next step)
codes_0.npy: VQ codes (used as prompt tokens below)

# Step 7: Generate Semantic Tokens (Text2Semantic Inference)
python3 fish_speech/models/text2semantic/inference.py \
  --text "Dies ist ein Beispielsatz für Sprachsynthese." \
  --prompt-text "amused" \
  --prompt-tokens "codes_0.npy" \
  --no-iterative-prompt \
  --compile

This creates:
temp/codes_0.npy: semantic tokens for final decoding

# Step 8: Decode to Synthesized Audio
python3 fish_speech/models/dac/inference.py \
  -i temp/codes_0.npy \
  -o output.wav \
  --checkpoint-path checkpoints/openaudio-s1-mini/codec.pth \
  --device cuda

This creates:
output.wav: your synthesized speech audio

# Step 9: Play and Save the Output
play output.wav
