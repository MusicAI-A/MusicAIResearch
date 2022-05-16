# #1 FastSpeech: Fast, Robust and Controllable Text to Speech

논문 URL : [fastspeech](https://arxiv.org/pdf/1905.09263.pdf)

## FastSpeech 요약

FastSpeech는 Feed-Forward Transformer(FFT) block들을 이용하여 구성되었고 Length Regulator가 duration predictor를 이용하여 
입력으로 들어오는 각 phoneme에 대한 duration을 예측하고 phoneme sequence의 hidden sates를 mel-spectrogram sequence에 맞게 
조정함으로써 길이 조절이 가능한 non-autoregressive TTS 모델이다.

- **Autoregressive TTS 의 한계점**
    1. 느린 inference 속도 : CNN, Transformer 기반의 TTS일지라도 autoregressive 모델은 긴 문장이 주어지면 합성 속도가 느리다. 
    2. 합성음이 robust 하지 않음 : skipping, repeating과 같은 문제 발생
    3. 합성음을 조절할 수가 없음 : autoregressive 모델은 하나의 mel-spectrogram 밖에 만들어 낼 수 없음  
    
    ![#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled.png](#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled.png)
    
- **Contribution**
    1. mel-spectrogram을 병렬로 만들어 합성 속도가 매우 빠르다.
    2. duration predicitor는 음소(phoneme)와 mel-spectrogram 사이에 강한 alignment를 보장하여 생략, 반복 단어의 비율을 줄인다.
    3. length regulator를 이용하여 쉽게 음성의 속도를 조절할 수 있다.
    
    ![#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%201.png](#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%201.png)
    
- **Structure**
    
    ![#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%202.png](#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%202.png)
    
    - Feed-Forward Transformer
        1. Feed-Forward Transformer (FFT) 는 Transformer의 self-attention과 1D convolution 기반 
        2. FFT는 FFT block들로 이루어져 있으며 phoneme side에 N개 mel-spectrogram side에 N개 
    - FFT Block
        1. self-attention 과 1D Convolution으로 이루어져 있음 
        2. self-attention network는 multi-head attention으로 이루어져 있음 
        3. Transformer에서 2층의 dense network를 사용한 것과 달리 2층의 1D convolution layer를 사용하였고 activation function은 ReLU 를 사용
        4. 1D convolution을 사용한 것은 인접한 hidden state들은 character/phoneme과 mel-spectrogram sequence에 대하여 매우 밀접한 관련이 있기 때문
    - Length Regulator
        1. 일반적으로 phoneme의 길이가 mel-spectrogram의 길이보다 짧음
        2. phoneme sequence의 hidden state들을 duration에 따라 d배 해줌
        3. $H_{pho}=[h_1, h_2, h_3, h_4]$ : phoneme hidden states
        4. $D=[2,2,3,1]$ : phoneme duration sequence
        5. $H_{mel}=[h_1,h_1,h_2,h_2,h_3,h_3,h_3,h_4]$ (α=1)
        6. 여기에 α 값을 적용하여 $D$에 곱하고 반올림하여 duration을 조정 
    - Duration Predictor
        1. 2층의 1D convolutional network with ReLU
        2. layer normalization
        3. dropout layer
        4. 마지막 linear layer 
        5. phoneme side 다음에 붙어서 jointly 학습 
        6. true duration은 autoregressive transformer TTS에서 구하였지만 추후 연구에서 사용하지 않을 예정이라 자세한 내용은 생략 
        7. 해당 구절의 의미를 아직 모르겠음 
        
        We predict the length in
        the logarithmic domain, which makes them more Gaussian and easier to train.
        
- **Experiment**
    - Datasets
        1. LJSpeech
        2. 13,100 샘플을 train / validation / test 를 12,500 / 300 / 300로 나누어 진행
        3. grapheme-to-phoneme conversion tool은 INTERSPEECH 2019의 논문 Token-Level Ensemble Distillation for Grapheme-to-Phoneme Conversion([https://arxiv.org/abs/1904.03446](https://arxiv.org/abs/1904.03446)) 을 이용. 해당 내용은 참고하여 English TTS를 진행할 때 참고 
        4. frame size = 1024, hop size = 256
    - Model Configuration
        1. phoneme side와 mel-spectrogram side 각각 6개의 FFT block 사용 
        2. phoneme vocabulary size는 문장 부호 포함 51 
        3. phoneme embedding size는 384 
        4. attention의 head는 2개 
        5. FFT Block 의 2층의 convolution layer 에서의 1D convolution의 kernel size는 3
        6. 첫번째 layer에서는 384 → 1536, 두번째 layer에서는 1536 → 384
        7. 마지막 linear layer에서 80차의 mel-spectrogram을 생성 
        8. duration predictor의 kernel size는 3, dimension은 각각 384  
        9. autoregressive transformer TTS는 사용하지 않을 예정이라 패스
- **Results**
    - Audio Quality
    
    Teacher 모델인 Transformer TTS와 준하는 수준의 MOS를 보임 
    
    ![#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%203.png](#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%203.png)
    
    - Inference Speed
    
    mel-spectrogram을 만들어내는 속도는 269.4배가 빠르고 WaveGlow를 결합하여 음성을 만들었을 때 38.30배가 빨라진다. 
    
    ![#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%204.png](#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%204.png)
    
    그림 (b)와 같이 Transformer TTS는 길이가 길어질수록 inference time이 완전 linear하게 증가하고 그 값과 변화 폭이 크지만 FastSpeech의 경우 inference time이 상대적으로 매우 적게 소요되며 mel의 길이에 따라 변화 폭이 크지 않다. 
    
    ![#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%205.png](#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%205.png)
    
    - Duration Control
    
    ![#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%206.png](#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%206.png)
    
    - Breaks Between Words
    
    인접한 단어들 사이에 break word를 적절히 넣어주면 prosody가 향상됨
    
    ![#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%207.png](#1%20FastSpeech%20Fast,%20Robust%20and%20Controllable%20Text%20t%2092f079d035184123a321160435e8089c/Untitled%207.png)
