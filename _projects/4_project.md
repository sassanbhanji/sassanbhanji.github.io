---
layout: page
title: Does ChessGPT understand Chess?
description: We used mechanistic interpretability to evaluate ChessGPT's understanding of the game
github: https://github.com/sb-2700/train_ChessGPT
importance: 4
category: work
---

This is a write-up of a mini-project I worked on with Mani Ipchi, Prince Kumar and Yusuf Kungdol as part of the Alignment Research Bootcamp Oxford (ARBOx3), building on prior interpretability work on ChessGPT. Credit to Yusuf for the write-up.

Adam Karvonen trained a GPT model to predict the next character in chess games written in PGN notation (e.g. 1.e4 e5 2.Nf3 Nc6...). The model was never explicitly given the rules of chess, yet it learned to play at around 1300 Elo and developed internal representations of the board state, as demonstrated through linear probing experiments.

Jon Kutasov and David Steinberg then investigated how ChessGPT represents "check" in an ARENA capstone project. They found something concerning: the model relied heavily on the '+' symbol in PGN notation rather than computing whether the king was under attack from piece positions. When they removed '+' from check positions, almost none of the SAE features that were good "check" predictors activated, and the model had nearly a 40% chance of predicting an illegal move.

Our question was: can we fix this through fine-tuning?

What is ChessGPT?
ChessGPT is an 8-layer transformer trained on around 16 million chess games in PGN format. Tokenisation is at the character level, so each token is a single character: letters, numbers, spaces, and special symbols like '+' for check and '#' for checkmate. The model's objective is next-character prediction.

The key issue is that every check in the training data was marked with '+'. This creates a strong correlation: the model sees millions of examples where '+' appears and the subsequent legal moves are constrained (you must move the king, block, or capture the attacker). The model could easily learn "if I see '+' nearby, certain moves become illegal" without ever learning to compute checks from piece positions.

The prior work suggested this is exactly what happened. Our goal was to remove this shortcut through fine-tuning on data without '+' symbols, forcing the model to learn check detection from the actual board state.

Does ChessGPT have an understanding of ‘checks’?
We took a dataset of chess games, stripped all '+' symbols from the PGN strings, and fine-tuned the model on 10,000 of these modified games. We then evaluated both the baseline and fine-tuned models on legal move prediction in check positions, testing with and without the '+' symbol present in the input.

The baseline model achieves 99.3% legal move accuracy when the '+' symbol is present in the input. This is expected: the model sees the '+' and knows to constrain its predictions to moves that address the check. But when we remove the '+' from the input, accuracy crashes to 57.6%. Nearly half the moves become illegal. The model genuinely does not know it is in check without that symbol telling it so. This confirms the findings from Kutasov and Steinberg, and gives us a clean baseline to improve upon.

After fine-tuning on 10,000 games without '+' symbols, the picture changes substantially. The fine-tuned model achieves 97.9% accuracy with '+' present (a small drop from 99.3%, which makes sense since we trained it without that symbol) and 92.2% accuracy without '+'. The gap between the two conditions shrinks from 41.7 percentage points to just 5.7 percentage points. This is an 86% reduction in the performance gap, suggesting the model now has something closer to genuine board understanding rather than relying on a notation shortcut.

We also measured behavioural consistency: does the model predict the same move whether or not '+' is present? For the baseline, same-move prediction was only 43.2%, meaning the model's output was highly dependent on whether it saw the '+' symbol. After fine-tuning, this increased to 63.0%. The model's predictions are now more stable across both conditions.

The interpretation is that training on 16 million games with '+' present hardcoded a circuit that detects '+' in the input and uses that to constrain legal move predictions. Fine-tuning without '+' forced the model to develop an alternative pathway that actually computes checks from piece positions. The model learned to do the thing it was supposed to be doing all along.

What about ‘check-mate’?
We ran similar experiments for checkmate detection. In PGN, checkmate is indicated with '#' instead of '+'.

We took 1,000 checkmate positions and tested whether the model could predict '#' at the end of the move. With both '#' and '+' removed from the input, accuracy was 72.9%. With only '#' removed (keeping '+'), accuracy improved to 79.6%.

The error distribution is revealing. Of all incorrect predictions, 99.5% were the model predicting '+' instead of '#'. Only 0.5% were other characters (typically spaces).

This tells us the model understands the king is under attack but cannot reliably distinguish check from checkmate. To detect checks, you only need to verify if any enemy piece attacks the king's square. Checkmate requires verifying that the king cannot escape to any safe square, no piece can block, and no piece can capture the attacker. The model has partially learned the first computation but struggles with the more complex second one. It knows something significant is happening to the king, but not the severity of the situation.

Effect of Injecting fake ‘checks’
We also tested the opposite direction: what happens if we add '+' symbols where they should not be? We injected random '+' characters into game strings at non-check positions and fine-tuned a model on this corrupted data. We then measured legal move accuracy on normal positions. In both conditions tested (keeping real '+' or removing all real '+'), legal move accuracy dropped to around 50-51%, regardless of whether fake '+' symbols were present or not.

A 50% legal move accuracy is essentially broken. For context, a functioning chess model should approach 100%. Injecting fake '+' symbols and training on this corrupted data completely destroys the model's ability to play chess. This makes sense: if '+' appears randomly and does not correspond to actual check, the model cannot learn any meaningful relationship between the symbol and the game state. It just learns noise.

This is a useful negative result. It confirms that '+' carries real information for the model, and that the model's reliance on this symbol is deep enough that corrupting it breaks everything. You cannot just add random check symbols and expect the model to figure it out.

Other interesting findings
We ran preliminary experiments on capture notation ('x' in PGN, e.g. Nxe5). The baseline model shows similar dependency: 99.1% legal move accuracy with 'x' present, dropping to 89.6% without (a 9.5pp gap, smaller than the 41.7pp for '+').

However, fine-tuning without 'x' produced unexpected results. After fine-tuning on 1,000 games without 'x', accuracy dropped to around 44% in both conditions. Fine-tuning on 10,000 games gave similar results (around 49% with 'x', 45% without). Unlike '+' fine-tuning, which successfully taught check detection, 'x' fine-tuning broke the model's ability to play chess entirely.

One interpretation is that 'x' is more fundamental to how the model processes move sequences than '+' is. The 'x' character changes the structure of the move notation itself (compare Ne5 to Nxe5), whereas '+' is appended at the end. Removing 'x' might corrupt the model's understanding of move syntax in a way that removing '+' does not. Another possibility is that our fine-tuning setup was not appropriate for captures. More investigation is needed here.

Next steps
The natural next step is mechanistic interpretability. Our results suggest there is a circuit somewhere in the model that detects '+' in the input and uses this to affect output logits. Finding this circuit would let us verify exactly how the model was "cheating" and confirm that the fine-tuned model uses a genuinely different mechanism. Other directions include evaluating the no-'+' model on checkmate prediction, investigating pawn captures specifically, and checking whether similar notation dependencies exist in other game-playing language models.

The broader takeaway is that superficial features in training data can mask whether a model truly understands the underlying domain. The '+' symbol is a clean example: redundant information that the model latched onto instead of learning the actual concept. This is probably happening in other domains too, and it is worth being careful about when evaluating what models have actually learned.

