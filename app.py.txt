import gradio as gr
from gtts import gTTS
import os
from pydub import AudioSegment

def text_to_speech(text, lang, voice_style):
    if not text.strip():
        return None, None

    # Generate base speech with gTTS
    tts = gTTS(text=text, lang=lang)
    file_path = "output.mp3"
    tts.save(file_path)

    # Load audio for modification
    audio = AudioSegment.from_file(file_path)

    # Apply simple voice-style effects
    if voice_style == "Female (High Pitch)":
        audio = audio._spawn(audio.raw_data, overrides={
            "frame_rate": int(audio.frame_rate * 1.2)
        }).set_frame_rate(audio.frame_rate)
    elif voice_style == "Male (Low Pitch)":
        audio = audio._spawn(audio.raw_data, overrides={
            "frame_rate": int(audio.frame_rate * 0.8)
        }).set_frame_rate(audio.frame_rate)
    # Default: Neutral voice

    final_path = "styled_output.mp3"
    audio.export(final_path, format="mp3")

    return final_path, final_path  # one for play, one for download

with gr.Blocks() as demo:
    gr.Markdown("# 🎙️ Lextone - Text to Audio Converter")
    gr.Markdown("Convert your text into natural speech with **different voice styles** and download the result.")

    with gr.Row():
        with gr.Column():
            text_input = gr.Textbox(label="Enter your text", placeholder="Type or paste text here...")
            lang_input = gr.Dropdown(choices=["en", "hi"], value="en", label="Select Language")
            style_input = gr.Dropdown(choices=["Neutral", "Male (Low Pitch)", "Female (High Pitch)"], value="Neutral", label="Select Voice Style")
            generate_btn = gr.Button("🔊 Generate Audio")
        with gr.Column():
            audio_output = gr.Audio(label="Generated Audio", type="filepath")
            download_output = gr.File(label="Download Audio")

    generate_btn.click(fn=text_to_speech, inputs=[text_input, lang_input, style_input], outputs=[audio_output, download_output])

demo.launch()
