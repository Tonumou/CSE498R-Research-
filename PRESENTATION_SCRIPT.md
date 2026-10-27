# CSE498R Final Presentation — 10-minute script

**Proactive Bitrate Adaptation for Resilient Cloud Gaming over Unstable 5G Networks**
Speakers: Tamanna (0:00–3:20) · Sanin (3:20–6:40) · Tashfiq (6:40–10:00). About 1,400 words; time one full run-through.

---

## Speaker 1 — Tamanna

**[Slide: Title] (0:00–0:35)**
"Good morning, respected faculty members. I am Tamanna Akter Mou, and with me are Sanin Tabassum and Tashfiq Mahmud Niloy. Our directed research, supervised by Rifat Ahmed Hassan, is titled *Proactive Bitrate Adaptation for Resilient Cloud Gaming over Unstable 5G Networks*. In one sentence: we trained a neural network to see a 5G throughput drop coming, so a cloud game can lower its video quality **before** the player feels lag, not after."

**[Slide: Background and problem] (0:35–2:00)**
"First, what is cloud gaming? In services like Xbox Cloud Gaming or GeForce Now, the game runs on a powerful remote server. The server renders every frame and streams it as video to your phone, and your button presses travel back. Anyone with a cheap device can play a high-end game, but only if the network is fast and stable.

5G millimeter-wave looks perfect for this, because it can deliver gigabits per second. But mmWave is fragile. A building, a bus or even your hand can block it. When you move between cell towers, which is called a handover, throughput can crash within a second. *[point at Fig. 4]* This is a real driving trace from our dataset. It jumps from about 900 Mbps to almost zero within seconds.

Netflix and YouTube hide these drops with a large buffer. Cloud gaming can't do that, because every frame depends on what the player just pressed. The delay from your input to the result on screen is called Motion-to-Photon latency, and it must stay within tens of milliseconds.

Today's Adaptive Bitrate algorithms, or ABR, are **reactive**. They adjust only after a drop has already happened, and for a game that means the frame has already frozen. So our research question was: can we **predict** the drop and adapt before it reaches the player?"

**[Slide: Literature review and gap] (2:00–2:55)**
"We reviewed twelve recent works. SABR combines behavior cloning with reinforcement learning, but it still chooses the bitrate from past network states. Reminis is proactive, but it works at the transport layer on TCP's congestion window, not on the game's video. EnDASH-5G uses an LSTM to predict throughput, but to save battery, not to protect gaming latency. SafeSABR adds a safety auditor, but it relies on backward-looking statistics.

So the gap is this: none of the work we found uses real 5G radio telemetry and forward-looking prediction to drive bitrate decisions in the application, specifically for cloud gaming."

**[Slide: Objectives] (2:55–3:20)**
"We had three objectives: predict 5G throughput from real measurements, use that prediction to choose video quality proactively, and measure how handovers affect throughput. Our long-term vision is a reinforcement learning agent, A3C with LSTM layers. In this phase we built and validated its foundation: the predictor and a rule-based adaptation policy. Sanin will now explain how we built it."

## Speaker 2 — Sanin

**[Slide: Dataset and preprocessing] (3:20–4:20)**
"Thank you. We used the Lumos5G dataset, published at ACM IMC 2020 by the University of Minnesota. It has 68,118 measurements, one per second, from Verizon's commercial mmWave network. They were collected on a 1.3-kilometre loop in downtown Minneapolis while walking and driving. Each row has download throughput measured with iPerf, LTE and 5G signal quality, device speed and the serving tower ID.

From 19 columns we picked seven features: throughput, moving speed, LTE RSRP and RSRQ, and 5G RSRP, RSRQ and SINR. Signal quality gives the model context beyond throughput alone. For example, a falling SINR can warn that a drop is coming. When the phone fell back to 4G, the 5G columns were empty. That was 23.7% of rows. We filled those with zero, which lies outside the valid dBm range, so the model can recognise 'no 5G'. Finally, we scaled every feature to the range 0 to 1."

**[Slide: Pipeline and LSTM model] (4:20–5:30)**
"Our pipeline has four stages. One: clean and normalize the data. Two: build sliding windows. Each input is the last 10 seconds of all seven features, and the target is the next second's throughput. That gave us 68,108 sequences. Three: split 80/20 **in time order**, giving 54,486 training and 13,622 test sequences. We did not shuffle, so the model never sees the future while training. Four: the LSTM predicts, and its prediction picks the bitrate.

Why an LSTM? Throughput is a time series. The LSTM's gates let it remember useful patterns, like a gradual fade, and forget noise. Our model has two LSTM layers with 64 hidden units and dropout of 0.2, then a 32-neuron dense layer and one output. We trained with the Adam optimizer at a learning rate of 0.001, MSE loss and batch size 256 for 50 epochs. All of it ran in PyTorch on an ordinary Windows laptop with no GPU."

**[Slide: Bitrate policy] (5:30–5:55)**
"The prediction is converted back to Mbps and mapped to five tiers. Above 500 Mbps gives 1440p at 6 Mbps. 200 to 500 gives 1080p, 100 to 200 gives 720p, 50 to 100 gives 480p, and below 50 gives 360p. The thresholds leave a lot of headroom on purpose, because mmWave can lose most of its capacity in a single second."

**[Slide: Challenges] (5:55–6:40)**
"We faced five main challenges. First, messy real-world data. A quarter of the 5G readings were missing, and the LTE SNR column was unusable because every value was a placeholder of about 2.1 billion. We dropped it. Second, extreme volatility. Throughput ranges from 0 to 1,920 Mbps, and its standard deviation is almost as large as its mean. Third, data leakage. A random shuffle would have given falsely good results, so we kept the split chronological. Fourth, hardware. We trained on CPU only, so we kept the model compact. Fifth, scope. The dataset has no latency measurements, so we could not yet build the full latency-based reward for the A3C agent. Tashfiq will now present our results."

## Speaker 3 — Tashfiq

**[Slide: Prediction accuracy] (6:40–7:25)**
"Thank you. On the 13,622 test sequences the model never saw during training, our 50-epoch model achieved an MSE of 0.0074, an RMSE of 0.086 and an MAE of 0.053. These are on the normalized scale. Going from 20 to 50 epochs improved MSE by about 3%, and the loss curve converges smoothly without overfitting. *[point at Fig. 7]* The red prediction tracks the blue actual throughput closely, including the large peaks and drops. The biggest errors are at sudden collapses, which are the hardest events to predict on mmWave."

**[Slide: Baselines and mobility] (7:25–8:10)**
"We compared against two baselines. A 10-step moving average lags behind every change, and our LSTM cut its MSE by more than 60%. The Last Value predictor simply repeats the latest measurement, which is roughly what reactive ABR does. It comes closer, but the LSTM still has lower MSE and RMSE, so it makes fewer **large** errors. Large errors are exactly what cause stutters in a game. By mobility, driving had 42.5% lower error than walking, because walking paths have more turns and more blockage from buildings. So pedestrians in dense cities benefit most from prediction."

**[Slide: Bitrate and handover results] (8:10–8:55)**
"Applied to the test set, our policy kept quality at 720p or above in **88.9%** of cases. It fell to 360p only 3 times out of 13,622. Stability matters just as much to a player. Adapting directly to the actual throughput changes quality in 27.87% of time steps. Our LSTM policy changes it in only **19.34%**, which is about 30% fewer switches and less visible flicker.

For handovers, we found about 1,370 tower transitions: 881 while walking and 489 while driving. In the five seconds after a handover, average throughput was about 85 Mbps lower than in the five seconds before. That is exactly the kind of drop a proactive system should prepare for."

**[Slide: Innovation] (8:55–9:25)**
"So what is new in our work? First, we move bitrate adaptation from reacting to predicting, in the application and specifically for cloud gaming. Second, we use the phone's own radio signals as inputs, not just past throughput. Third, we train and test on real commercial mmWave traces, not synthetic simulation. Fourth, we evaluate stability and handover impact as well as accuracy. Finally, the model is light enough to run on a CPU, which makes deployment on an edge server realistic."

**[Slide: Limitations and future work] (9:25–9:45)**
"We also want to be honest about the limitations. We predict only one step, one second, ahead. The policy uses fixed thresholds, the dataset has no latency data, and our testing is trace-driven rather than on a live network. Next, we plan multi-step forecasting, an A3C agent that uses the LSTM's prediction as input, testing on the Irish 5G and MN-Wild datasets, and deployment on a mobile edge server."

**[Slide: Conclusion] (9:45–10:00)**
"To conclude: 5G mmWave is fast but unstable, and cloud gaming cannot hide that behind a buffer. Our LSTM keeps 88.9% of gameplay at HD or better while switching quality 30% less often than reactive adaptation. Prediction, not reaction, makes cloud gaming resilient. Thank you, and we are happy to take your questions."

---

## Likely questions, with answers

1. **"Last Value has a lower MAE than your LSTM. Is the LSTM just copying the last value?"**
   One second ahead, repeating the last value is naturally hard to beat. But the LSTM wins on MSE and RMSE, which punish large misses, and large misses cause stutters. Last Value also switches quality 27.86% of the time, against 19.34% for the LSTM. The LSTM's advantage should grow with longer forecasts, which is the next step.
2. **"Where is the A3C agent from your objectives?"**
   This was phase one. The predictor is the input the agent needs, and the dataset has no latency data to build its reward.
3. **"You promised a 100–300 ms warning."**
   The dataset records one sample per second, so one second is the finest we can resolve. Sub-second forecasting needs finer data.
4. **"Why need 500 Mbps for a 6 Mbps stream?"**
   Headroom. Throughput can lose about 90% within a second, and the prediction error is about 166 Mbps. The thresholds also roughly follow the data, whose median is 424 Mbps. A learned policy would tune them.
5. **"What is MAE 0.053 in Mbps?"**
   Multiply by the 0–1,920 Mbps range: MAE is about 102 Mbps and RMSE about 166 Mbps.
6. **"Is filling missing 5G values with 0 valid?"**
   Zero dBm is outside the valid −140 to −44 range, so it works as a "no 5G" flag. A cleaner option is an explicit 5G-connected feature.
7. **"Show us the code for the baselines and the handover analysis."**
   Open `CloudGaming_Full_Analysis.ipynb`: Section 8 (baselines), 9 (walking vs driving), 10–11 (bitrate tiers and switching), 12 (handover).
8. **"Your notebook's LSTM numbers aren't exactly the report's. Why?"**
   Every training run starts from random weights, so the LSTM numbers move slightly. We retrained the 50-epoch model with four different seeds. MSE stayed between 0.00743 and 0.00758 (mean 0.00751), always better than Last Value (0.00821) and the moving average (0.01984). Switching stayed at 18.6–18.7%, and 720p-or-higher at 86.5–87.5%. The baselines and the handover numbers don't involve training, and they match the report exactly.
9. **"How exactly did you measure the handover drop?"**
   A handover is any change of `tower_id` between consecutive seconds. We compared the mean throughput over the 5 seconds before the change (386.99 Mbps) with the mean over the handover second and the 5 seconds after it (301.75 Mbps).
