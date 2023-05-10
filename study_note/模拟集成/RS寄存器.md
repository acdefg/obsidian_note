![200](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230510092611.png)

![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230510092629.png)

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20230510092538.png)

注意到
当 R=1 时，输出为 0，故 R 又称为直接置“0”端，或“复位”端（reset）
当 S=1 时，输出也为 1，故 S 又称为直接置“1”端，或“置位”端
当 R=S=0 时，输出保持不变（很重要的特征！保证了 RS 同时为 0（断电）后，电路输出能够保持不变）
注意！！！RS 不能同时为 1
如果 RS 同时为 1，那么根据电路图可以推导出两个输出全为 0，有人可能会说这有什么大不了，但是接下去当 RS 同时变为 0 的时候，问题来了！！！
由于 RS 不可能同时变为 0（电路时延不可能完全相同），那么就存在先后问题，就会给电路带来不确定性！因为我们不知道是谁先变成 0，就更不知道输出会变成什么样！

