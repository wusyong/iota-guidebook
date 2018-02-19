Tangle 上的平衡：容我嘗試解釋...

從  IOTA 初期以來，就有些關於預設 tip 選擇演算法並非強制性的問題受到關注，沒有辦法確認一個節點真的就是使用推薦的 MCMC 演算法，或者它也可以偷改其中的選擇來取得更大的利益。如同白皮書所討論到的，整個系統要正常地運作的確需要 tip 選擇要夠隨機，一個節點選擇附加交易的地方要在所有 tip 之上，而且相對機率要趨近相同。欲了解隨機的重要性，我們假設有節點總是按照一些衡量標準選擇最好的兩個 tips。由於還有其他非常多的節點而且交易的流量相當地大，這些選擇這兩個所謂「最好」的 tips 勢必成為「競爭」。在這些競爭者中，只有一些人有幸能選中（他們的 tips 也因此成為最佳）而其他人的則會因此被遺棄。

![](https://cdn-images-1.medium.com/max/800/1*Qs_KFwcXxXKuoERjfJ5xsw.jpeg)

這是我們想要避免的狀況：中間黑色為一條非常細幾乎像是區塊鏈的圖形，大部分綠色的交易很可能永遠變成孤兒，而標示為藍色的是最近的 tips，他們很可能會步上綠色交易的後塵。

所以自私節點採取「貪婪」的策略的確會是個顧慮，最終可能導致上圖的情況發生，而整個系統失敗。至於我們該如何探討這個問題會不會發生，我們可以先看一下一些理論。

當一個由兩個或多個玩家組成的非合作博奕遊戲中，如果某情況下無一參與者可以通過獨自行動而增加收益，則此策略組合被稱為奈許均衡點，如同著名的囚徒困境博奕般。而奈許均衡點只會發生在 Pareto-optimal 的結果上。在沒有其他可能的結果的情況下，如果沒有讓至少一名玩家變壞，任何玩家都會變得更好。

在一個雙人或多人的非合作賽局中，若沒有任何人能採取與目前策略不同的獨自策略獲得更多收益的話，則稱此為納許均衡點（Nash Equilibrium）。在特定的情況下，納許均衡點會讓所有人的收益比合作賽局獲得的收益還要來得慘，最有名的例子就是囚徒困境。而其他情況下，納許均衡點會發生在帕雷托最優（Pareto-optimal）上，不會有人因此受益並讓其他人受害。因為 IOTA 是去中心化、分散式且無權限的節點網路，必須假設系統中有一部份的節點在進行非合作博弈賽局，只進行會讓自己利益最大化的選擇。因此在自私（貪婪）節點必定存在的狀況下，了解此賽局中的平衡狀況是非常重要的。它會讓整個系統都變得自私自利嗎？還是在帕雷托效率下會出現納許均衡點？在這種情況下會出現納許均衡點嗎？

Almost 70 years ago John Nash proved the existence of equilibria for the case of finite number of players each having finite sets of possible choices, in the situation when the outcome is determined (i.e. nonrandom) by the players’ choices. This last condition, however, does not hold for the Tangle, since, at the time the transaction is issued, it is not known when (and whether) it will be confirmed. So, in the paper Equilibria in the tangle, we present a rigorous proof of the existence of a Nash Equilibrium in the non-cooperative game where some fraction of the nodes choose a greedy tip selection strategy to minimize their cost (for example, the time it takes for their own transactions to be approved). We also proved that, given the number of nodes in the network is large, all Nash equilibria are “almost symmetric”, in the sense that the costs of all nodes are roughly the same, and this, in its turn, permits us to assume that all nodes can adopt the same strategy. We need the latter for the simulations, since trying to simulate many nodes with a plethora of different strategies would be unfeasible.

All the above theory, however, does not answer the question we posed before: will the selfish nodes “destroy” the network? To answer this question we need to see how the Nash equilibria look like, but, due to the overall complexity of the system, it seems to be too difficult to obtain a purely analytic solution, even in a (relatively) simple situation when a selfish node chooses (with some probability) the above “greedy” strategy of approving two “best” tips. However, it is still possible to understand why that “naive” greedy tip selection strategy would not be a Nash Equilibrium in the noncooperative game played between greedy nodes.

![](https://cdn-images-1.medium.com/max/800/1*qvNmyzQijU3PpMYvYtaxGg.jpeg)

Why the “greedy” tip selection strategy will not work (the two “best” tips are shown as larger blue circles). Many selfish nodes attach their transactions to the two “best” tips believing that by choosing these tips their transactions will be more likely to be chosen by subsequent transactions. As a result, the “neighborhood” of these two tips becomes “overcrowded”: there is so much competition between the transactions issued by the selfish nodes, that the chances they are selected for approval by subsequent transactions actually decreases and they all lose.
Simulations were performed to validate the above intuition, and they showed that it was correct indeed (in a subsequent blog post we will describe these simulations in a much more detailed way). Moreover, in the Nash equilibrium the system was nearly as efficient as in the “fully cooperative” regime (no selfish nodes) which could suggest a Pareto-optimal Nash Equilibrium. As a concluding remark, we observe that, given that the selfish nodes still have some extra costs (for example, they need to calculate the exit distribution of some Markov chain on a very large state space, which can be computationally expensive), they will have little or no incentive to bother following any fancy tip selection strategies instead of the default one.

For Part 2 of this Series:
https://blog.iota.org/equilibria-in-the-tangle-let-me-try-to-explain-part-2-6dcc8e7c0ad8
