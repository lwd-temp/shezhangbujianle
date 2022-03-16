借助世界上最大的单体中文NLP大模型(浪潮 源1.0 https://air.inspur.com/home ），我们做出了一个可以跟人类玩“剧本杀”的AI……


# 无图无真相
我们为本项目特别改编了一个微型线上剧本杀剧本，本子有五个角色，分别由五名玩家扮演，但我们每场只会召集四个玩家，并在他们不知情的情况下，派出AI扮演剩下的那个角色。

为了尽量避免引起玩家怀疑，我们不想让用户额外下载APP或者登录一个网页。对于绝大部分中国用户而言，微信自然就是最自然的交流方式，也是“最真实”的交流方式，为此我们需要打造一个“微信虚拟人”，
很幸运的，我们发现了wechaty这个神器（https://github.com/wechaty ）。

本着精益求精、做假做到底的原则，编导组精心策划了这个账号的昵称、头像，甚至每场游戏前我们还会紧扣时事的准备近三天的朋友圈内容……
![img](）

整体剧情并不复杂，讲的是某高校社团中五个骨干成员因为一件事情牵涉到各自利益而产生的种种勾心斗角。玩家要做的也非常简单，就是想方设法、拉帮结派的说服其他人接受自己的主张……不过我们这次对原作做了比较大的改动，剧作中AI所扮演的角色（蔡晓）受控于某邪恶的科技巨头（“北极鹅”公司），她要帮助“北极鹅”实行一个庞大的阴谋，而这个阴谋其实笼罩了所有人……坦率的说，从游戏角度，这个角色的难度还挺高，承担着推动剧情的作用，并且游戏机制设定最后所有的疑点矛头都会指向她，如果在现实的剧本杀游戏中，这个角色也应该是由DM扮演，而非普通玩家，当然这也就大大增加了对AI的考验。

下面四幅图展示了AI的实际表现效果（游戏中会要求玩家更改群昵称，而这里为了保护玩家隐私，也为了方便大家理解，我们直接把玩家的微信昵称备注为了角色名）。

### 谭明VS蔡晓（AI）
剧情中谭明为了实现自己的目的，不择手段的策划了一个诡计，并计划私下与蔡晓达成联盟，然而他不知道的是蔡晓其实在下一盘更大的棋，正想借他的诡计实现自己的阴谋……所以AI对谭明的策略就是可劲儿的忽悠他并想方设法利用他。实际表现中，AI很好的贯彻了这个思路，甚至发挥想象力的使用了色诱绝技……坦率的讲，这招也极大的超出了我们的预料……
![img](https://github.com/WukongZeming/shezhangbujianle/blob/main/assets/%E5%B1%8F%E5%B9%95%E5%BD%95%E5%88%B62022-03-13%2014.48.44.gif)

### 孔墨VS蔡晓（AI）
### 李超VS蔡晓（AI）
### 孙若VS蔡晓（AI）
### 蔡晓（AI）在公聊（房间）

# 核心功能——“目的性对话”端到端生成方案
“剧本杀”本身就是一种用户与用户之间充分博弈的游戏，玩家之间的对话可能更是无穷无尽，这种情况下使得传统的规则式或“词槽式”“目的域对话”方案均行不通，同时因为玩家之间的对话都是带有目的性的，尤其是AI所扮演的这个角色还承担着部分推动剧情的DM功能，所以“开放域”对话也不适合。

本项目所使用的NLP大模型浪潮源1.0是一种生成式预训练模型（GPT），其使用的模型结构是Language Model（LM），类似于openAI的GPT-3，但是与GPT-3不同，源1.0更加擅长的是零样本（Zero-Shot）和小样本（Few-Shot）学习，而非目前更多模型所擅长的微调试学习（finetune）。从实际应用效果来看也确实如此，在2~3个，甚至1个合适example的示范下，模型可以很好的理解我们希望实现的“对话策略”，仿佛具有“举一反三”的能力，但是如果没有example的话，那么模型的生成则非常不靠谱，甚至会出现答非所问的情况。因此，本项目的关键就在于如何针对用户的提问选择适当的example供给模型。

最终我们采取的方案是：建立example语料库，然后针对每次提问从语料库中选择最贴近的三个example作为模型本次生成的few-shots。

实际实现中，因为除AI扮演的角色外游戏还有四个角色，而每个角色针对AI角色的提问又有两种可能：1、私聊中提问，2、公聊（群）里面的@提问，所以语料库被分装成8个TXT文件，程序会根据提问者以及问题来源（私聊还是公聊）去对应选择语料来源。这个机制的思路很简单，但是执行起来马上遇到的一个问题就是，如何从对应语料中抽取与当前提问最为相似的example？因为在实际游戏中，玩家可能的提问会很多，为了照顾到各种情况，我们准备的语料库也会涵盖很多问题（截止本项目发布时，8个语料库文件最少的问题量也达到了  个），显然不可能把这些问题都作为example在每次生成时输给模型。在这里我们用到了百度飞桨PaddlePaddle发布的预训练模型——simnet_bow，它能够自动计算短语相似度（https://www.paddlepaddle.org.cn/hubdetail?name=simnet_bow&en_category=SemanticModel ），基于百度海量搜索数据预训练，实际应用下来效果非常不错，且运算速度快，显存占用低。这里不得不特别表扬下百度飞桨，尤其是其中的预训练模型开放库paddlehub，里面有不少非常实用的“小模型”，本项目除了使用了simnet_bow外，还应用了paddlehub的另一个模型ernie_csc（https://www.paddlepaddle.org.cn/hubdetail?name=ernie-csc&en_category=TextCorrection ），它能够自动进行文本纠错，包括错别字更正、简单病句修改，本项目的游戏体验都是基于微信打字聊天，玩家难免出现手滑现象，所以这个功能就显得很有必要。 整体而言，百度飞桨语法简单、学习门槛低，paddlehub提供的预训练模型更是具有开箱即用、运行稳定、简单实用的特点，本项目同时使用simnet_bow和ernie_csc两个模型，显存占用也才1.5G，确实称得上是“产业级AI框架”，只可惜paddlehub提供的都是小模型，并没有这两年大行其道的GPT类大模型，技术上显得稍微落后了些，当然这也可能是我们使用的是免费版的缘故……

总之在simnet_bow的加持下，抽取最相似example的问题得到了圆满解决：程序接收到某玩家提问，会根据玩家角色和提问来源（私聊或公聊）匹配对应的语料库文件，然后玩家提问文本在ernie_csc处理后与语料库文件中的问题文本逐个进行相似度计算，之后按相似度排序，选择相似度得分最高的前三个连同语料库中的答案作为example提交源1.0 API进行生成，返回的text再通过wechaty连接AI账号登录的微信账号发出……

当然生成的prompt也是十分关键的，这一点上源1.0与GPT-3一样，因为这类模型生成的本质是续写，Prompt兼有任务类型提示和结果开头的作用，另外example也是需要通过prompt形式给到模型的，这样问题就变得稍微复杂了起来，盘古大模型群里面有网友甚至说，他感觉GPT-3的prompt书写本身就相当于一门编程语言了……不过这次浪潮团队的技术支持可谓“暖男级”贴心，针对API提交团队给出了详细的、质量极高的范本代码（可以说也是我见过的大模型开源项目中给到的质量最高的示例代码），所以我们在API提交这块就直接用了浪潮团队的代码，这极大的降低了工作量，本项目代码库中的__init__.py、inspurai.py、url_config.py这三个文件就直接来自浪潮的开源代码（https://github.com/Shawn-Inspur/Yuan-1.0/tree/main/yuan_api ）。

至此所有的工程问题已经基本都解决了，剩下的就是语料来源问题。实际上这也是最核心的问题之一。GPT类大模型生成本质是根据词和词的语言学关联关系进行续写，它是不具有人类一样的逻辑能力的，即我们无法明确告知它在何种情况下应该采用何种对话策略，或者该往哪个方向去引导，这一切都得靠example进行“提醒”。打个不恰当的比方，浪潮源1.0相当于一个很聪明，读了很多书的“学生”，但是它从小生活在一个与世隔绝的村子里，完全不了解城市里的各种套路，也从来没有玩过剧本杀，它能做的就是模拟、有样学样，但是它的绝技是学的极快……所以这个AI在游戏中的表现就直接取决于我们教的，或者说我们“演示”得如何。（好比张无忌如果不是碰到那么多世外高人，碰到的都是你我这样的凡夫俗子，他天资再高也不可能练出九阳神功）。

对于这个问题，团队最终采取了一个非常简单粗暴的方案：编导组除主编外每人负责一个角色（刚好四人），自己没事儿就假装在玩这个游戏，想象看会跟AI提什么问题，然后假设自己是AI扮演的角色，如何回答是最好的……初始语料文件好了之后，大家交换角色进行体验，每次体验后更新各自负责的语料库文件，之后公测也是一样，每轮之后编导组都会根据当场AI回答得比较差的问题进行语料库的完善和补充……为此我们在程序中增加了一个功能，即程序会把本场用户所有提问以及对应该问题抽取出的三个example问题的simnet_bow相似度得分和源1.0最终生成的回答问题，按语料库对应另存为8个文本文件，以便于编导们针对性每轮更新语料库。本项目目前开源提供的语料库是截止     ，公测   轮后的版本。可以说，这个机制是一种人类辅助式“在线学习机制”。

# 一种NLP大模型时代全新的“创作范式”
现阶段的NLP大模型虽然已经发展得远远超出人们预期，这些成就哪怕放到三年前可能都是不可想象的，但依然不可否认的是其能做的还是十分有限，主要原因在于两点：1、机器不具有哪怕是五岁小孩那样的逻辑能力，更别提人类交流中那种默认的对上下文、环境乃至话外之音的理解能力，一个典型的例子是，直到现在，任何NLP大模型对于“他、她、它”的正确引用都是十足的弱项（这一点源1.0还算是出类拔萃的了，但实践中错误使用的概率也不低）；2、生成的不可控性，这是神经网络本身性质决定的了。同样的对话条件，生成的结果可能会很不一样（但是每一个也都说的通），由此导致本项目的“精彩对话”其实很难复现，我这里也特别说明本文中的示例贴图都是我们事后使用demo程序“还原”的，而非实时生成，但这确实100%是由AI在某次游戏中生成的，我们未加任何调整（包括标点符号！）。

在这两点的限制下，剧本的选择、乃至游戏形式的选择就是十分关键的了，你必须充分发挥AI的特性又规避它的不足。我最开始策划这个项目是在春节前后，彼时我已经拿到源1.0的API权限，但愣是将近一个月的时间没想好该做什么，构思的方案要么太依赖规则，AI则显得“可有可无”，要么则太依赖生成，AI又“难堪重任”，我想可能这也是目前AI行业整体面临的问题吧，实验室出来的东西都挺振奋人心的，但真的要在实际业务中上，又实在硬伤太多，当然我们要用发展的眼光看问题，大模型才几年，神经网络也才几年……当然，本项目比较幸运的是最后想到了“剧本杀”这个方案，并且我们所改编的这个本子又非典型的“本格逻辑本”，而更加类似“角色扮演”，玩家之间的互动也主要就是互相提问、对话，这种情况下，AI生成的“不确定性”反而成为了一种优势，想想看，人类其实也一样，外部条件都一样，同一个人每次的决定、每次说的话可能也都不一样。而我们也充分考虑了各种“穿帮”的可能性，最后对AI所扮演的角色的剧情进行了彻底性的改编，这个改编甚至改动了整个剧本的故事背景，一个本意是用于大学生心理辅导的“岁月静好”本被我们活活改成了“赛朋博克风校园微恐本”：），而为了增加剧情效果，我们又参考了“规则怪谈”类文学作品，为AI增加了很多“朋友圈”和游戏后“返场”戏码（为了保证之后的用户体验，请原谅我这里就不详细透露剧情了,但是我已经在本项目中分享了全部人物读本和导演手册，见script文件夹）……可以说，本作是我们为AI量身打造的一个本子，整体创作过程是先有AI、再有剧本的，虽然如此，但最终所有的创意都很好的适配了AI的特性，而AI的不确定性也刚好为剧本增加了“戏剧化”成分，这不仅仅是说AI针对每个问题每次的回答策略都会不同，而是AI会很奇妙的随机增加一些“剧情”，这些剧情在逻辑自洽的前提下为本作提供了宝贵的随机性，可以说本作最终呈现给玩家的是一个“活的故事”，没有人可以玩到两次一模一样的……

我们相信这种**内容编辑与工程代码的混同创作**会是一种适应NLP大模型的崭新“创作范式”，它不仅仅可以应用于娱乐内容，其他领域诸如教育、客服、销售、政务、心理情感等涉及到“目的性对话”的领域均可以尝试。

因此，我们在本项目中也一并将最新版的语料文件进行开源，因为在这种“开发范式”下，这些语料文件本质上就是代码的一部分。

# 致敬
除综合使用了浪潮源1.0NLP大模型和百度飞桨的paddlehub预训练模型外，本项目涉及到的另一个关键组件是wechaty，这是一个可以几乎无缝对接包括微信在内一众主流IM软件的“神器”，如果没有它，我们将只能让用户通过某个网页或者APP与AI进行交流，这不仅极大的增加了开发工作量，也大大削弱了实际体验，甚至无法实现我们的关键创意：让AI悄无声息的潜入玩家中，让玩家最终获得“自己就在戏中”。

wechaty操控微信是通过所谓的“wechaty-puppet”实现的，wechaty社区目前提供多个wechaty-puppet方案，我们这次使用的是完全免费的puppet-xp方案。这是一个基于Windows微信客户端协议的方案，跟本项目非常适配，因为我们将主程序代码和微信客户端跑在一台电脑上，部署最省事，然后还完全免费！好比我写代码时最爱听一首歌在网易云音乐上的热评——既能听林忆莲，又能抖腿，你们还要啥？

不过也不得不说的是，puppet-xp目前的功能实在有限，仅能实现文本信息的收发。另外群聊中也没法获得玩家的群昵称，导致AI在群中的回复只能@玩家的微信名，但你在微信客户端上操作是默认@群昵称的，这其实是本作一个无法遮蔽的穿帮的地方。好在puppet-xp团队也在不停的升级开发，我们期待后续版本可以在免费的同时提供更多功能支持。现在NLP大模型都有往多模态发展的趋势，text2img甚至text2video都已经在路上，而这也是我下一个项目所将要尝试的功能。

然而在这里，还是要向@wechaty社区致敬！”好用到哭“——你们对得起这个评价！

也向爱写诗的laozhang（puppet-xp作者）致敬！感谢你把这么好的东西无私的奉献了出来！

致敬浪潮源1.0团队、百度飞桨团队，致敬github和csdn，本项目实现过程中没少在这两个地方抄代码（其实不止本项目了，我想每个程序员都一样吧），总之致敬开源社区，致敬所有开发者！

# 创作名单
### 主创&工程&内测、公测导演
大兄弟666 —— Forgive me for having to remain mysterious
### 编导组成员
李青玲 —— 上海交通大学媒体与传播学院

胡璟葳 —— 华东理工大学新媒体艺术学院

赵家宁 —— 上海交通大学媒体与传播学院

翡冷翠的小羊 —— 跟这兄弟网友好多年，从来不知道真名，就更别提其他的了……

编剧指导：韩飞

感谢所有参与公测的同学，希望没有把你吓得太过，虽然这也正是本作的目的之一 :smirk: ……

特别感谢华东理工大学图书馆的**吉久明教授**授予我们原作的改编权，同时还无偿贡献了配套的“角色选择心理测验“，通过16道是否题，就可以比较精确的匹配到更玩家性格特征最贴近的角色，从而让整个游戏的沉浸感更加强烈，个人认为，这倒是每一个剧本杀都值得引入的一个机制。

# 写在最后的话
做这个项目是有感于去年大热的各种虚拟人，我丝毫不怀疑这个赛道的潜在价值，虽然我们目前还没有办法统一而清晰的定义出作为人类最终归宿的“元宇宙”，但有一点是毋庸置疑的，即未来的元宇宙空间中，虚拟人数量将数倍于真人，因为只有这样，才能让我们每个人过上比现在更好的生活，但我也发现，在目前阶段，各路虚拟人在好看的皮囊方面已经愈发进步，然而在有趣的灵魂方面好像还都普遍欠缺，靠“中之人”驱动毕竟不是长久之策……而另一方面，自去年上半年我了解到NLP领域近两年来在生成式预训练大模型方面的长足进展后，也一直想看看基于这种大模型有什么可以实际落地的场景，就这样两个不同角度的想法合流成为了本项目的初衷……

最后，我选择将本项目全部代码开源，一方面在这个项目中我用了太多开源的资源，最后的项目成果不开源实在不地道；另一方面，也欢迎有高手和专业人士不断完善这个项目，尤其欢迎其他中文NLP大模型研究机构移植本项目代码，看看自家模型在这种“目的性对话”应用方向的潜力。如果有机会能够看到各路AI机器人互相玩剧本杀我想会是一个有趣的事情，而这种良性的比拼和竞争也必将对中文NLP大模型的发展有所裨益。
