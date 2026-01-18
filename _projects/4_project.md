---
layout: page
title: Does ChessGPT understand Chess?
description: We used mechanistic interpretability to evaluate ChessGPT's understanding of the game
github: https://github.com/sb-2700/train_ChessGPT
importance: 4
category: work
---

This project was done with Mani Ipchi, Prince Kumar and Yusuf Kungdol for our final project in ARBOx3.

What is ChessGPT?
- it is an 8-layer transformer model based on GPT-2 that was finetuned on 16 million chess games.
- It reads in PGN strings (;1.e4 e5 2.Nf3 ...) and predicts the next token (each character is a token)

We built on some previous work by Jonathan Kutasov that showed that if the '+' (which denotes check) is removed from the sequence of tokens, the model's ability to make legal moves drops from 99% to 60%. This is interesting because the check can be inferred from the board state, so this result might suggest that maybe ChessGPT doesn't really understand the board state and has just memorised that a '+' denotes check.

We investigated this by finetuning the model on a relatively small number of samples (10k) that didn't include the + notation. Our evals on this model showed that performance went back up to 97% legal moves. This could mean that the original ChessGPT had some circuit that 'hardcoded' what to do when there was a + token in the sequence.

We then calculated the KL divergence between the logits for an input with +s and one without +s. For the original ChessGPT this was 3.008, showing how omission of a + shifts the logits away significantly. The KL divergence was only 0.046 for our finetuned version which shows that the model does actually understand chess to a good level once the circuit that hardcoded checks was removed.

We carried out similar experiments on the 'x' token which denotes a take and '#' which denotes checkmate.

Further work on this would include using activation patching to find the transformer heads that are responsible for detecting the '+' token and using SAEs to identify features that activate on check.
