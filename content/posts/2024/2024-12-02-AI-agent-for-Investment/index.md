---
title: "AI Agent: The future of Investment research"
description: "AI Agent ứng dụng trong phân tích và đầu tư"
date: 2024-12-02
draft: false
math: katex
summary: "AI Agent là một hệ thống tự động và độc lập, có khả năng xử lý thông tin, ra quyết định và thực hiện các nhiệm vụ trong nhiều lĩnh vực. Bài viết mô tả cách xây dựng một đội ngũ phân tích đầu tư tự trị với các vai trò như phân tích kỹ thuật, phân tích thông tin, tư vấn đầu tư và xây dựng chiến lược giao dịch, sử dụng các công cụ để tự động hóa quy trình phân tích và ra quyết định. AI Agent có tiềm năng ứng dụng rộng rãi trong tương lai gần, tuy nhiên, việc tạo ra một agent đáng tin cậy vẫn là thách thức lớn."
---

![1.png](images/1.png#center)

## Concept

AI Agent, một topic hiện đang rất hot trên thế giới về lĩnh vực AI. 

Theo fpt.ai:

**AI Agent** là một hệ thống phần mềm hoặc phần cứng được thiết kế để thực hiện các nhiệm vụ một cách tự động và độc lập, nhằm đạt được những mục tiêu nhất định.

Một **AI Agent** có khả năng xử lý thông tin, đưa ra quyết định và thực hiện các hành động để tương tác với các điều kiện và hệ thống liên quan. Nhân sự [**AI tạo sinh**](https://fpt.ai/vi/bai-viet/generative-ai/) có thể áp dụng cho nhiều lĩnh vực, từ trợ lý ảo cho khách hàng đến hệ thống điều khiển tự động phức tạp, thậm chí là robot trong các môi trường thực tế.

Theo IBM:

An [artificial intelligence (AI)](https://www.ibm.com/topics/artificial-intelligence) agent refers to a system or program that is capable of autonomously performing tasks on behalf of a user or another system by designing its workflow and utilizing available tools.

Tóm gọn lại: AI agent là một hệ thống thông minh và autonomous (tự trị → tự suy nghĩ, suy luận, ra quyết định,…). Mỗi agent có thể thực hiện được các task mà người thiết kế giao. Và một điểm đặc biệt so với chatbot thông thường là người thiết kế cho thể đưa cho Agent các tools để tự sử dụng và các Agent phối hợp với nhau để hoàn thành mục tiêu cuối cùng.


---
Ví dụ, trong đầu tư, ngày xưa bạn có tool lấy giữ liệu từ database lên. Bạn sử dụng nó và tool trả ra output, bạn dùng output này cho vô AI. Giờ thì đã khác, bạn có thể đưa sẵn tool cho Agent, và các Agent này sẽ sử dụng chính cái tool này để thực hiện các task cần thiết, loại bỏ bạn ra trong quá trình “tự trị” của nó.

Nếu bạn chưa hiểu thì hãy tưởng tượng như sau.

Bạn là 1 financial analysis, trong quá trình phân tích, bạn sẽ luôn follow 1 số quy trình nhất định và sử dụng 1 vài tools nhất định.

**B1**: lên google search cổ phiếu HPG.\
Tại bước này, bạn sử dụng 1 tools là **google_search**

**B2**: download các bài báo về, lưu xuống database, sử dụng 1 tool sentiment calculator để tính điểm các bài báo này.\
Tại bước này, bạn sử dụng 1 tools là **sentiment_calculator**

**B3**: Viết report theo 1 khung nhất định (Thân bài, mở bài, kết bài) đi kèm các tool khác như: vẽ chart correlation với thị trường, tính toán các chỉ số tài chính ,..\
Tại bước này, bạn sử dụng 2 tools là **report_writing** và **metrics_calculator**.

Và, điều thú vị ở đây là, bạn chỉ cần chuẩn hoá các tools trên như là 1 function trong python và đưa cho Agent. Các Agent đã có thể thực hiện từ bước 1 đến bước 3 mà không cần bạn can thiệp vào bước nào. Ngoài ra, thông qua cơ chế Specialization và Multi-Agent system, bạn còn tăng được độ chính xác, hiệu quả và khả năng thích ứng của hệ thống tự trị này [(fpt.ai)](https://fpt.ai/vi/bai-viet/multi-agent-system/).

Và điều thú vị hơn nữa, là làm những thứ này, rất đơn giản, thông qua kĩ thuật prompt engineering.

## Xây dựng một hệ thống phân tích kĩ thuật, thông tin và tư vấn chiến lược đầu tư.

Hãy cùng nhau thử nghiệm đi xây dựng một team phân tích tư vấn như một phòng trong công ty chứng khoán nhé. Lưu ý, phòng tư vấn này “tự trị” - autonomous. Việc cần làm là đưa vào 1 mã cổ phiếu (ví dụ HPG), các AI Agent sẽ tự xử lý 100%.

Phòng tư vấn của ctck có mô hình tổ chức đơn giản nhất như sau:

- 1 bạn phân tích kĩ thuật (technical analyst)
- 1 bạn phân tích thông tin, báo (news analyst)
- 1 bạn tư vấn đầu tư (investment advisor)
- 1 bạn xây dựng chiến lược giao dịch (strategy advisor)

![2.png](images/2.png#center)

Cùng nhau xây phòng tư vấn nào. Trước tiên, để thực hiện các nghiệp vụ trên, ta cần các công cụ sau:

- Lấy giá cổ phiếu, tính các chỉ báo kĩ thuật
- Lên google search và thu thập các thông tin liên quan đến cổ phiếu ta muốn

```python
## Mình sử dụng gpt-4o-mini là model llm bên dưới
llm = ChatOpenAI(model="gpt-4o-mini")

## Tools
def load_price_and_ta(company_stock):
    query = f"SELECT * FROM eod WHERE symbol = '{company_stock}' order by timestamp DESC LIMIT 1000"
    host = 'xyz' (database link)
    response = requests.get(
        host + '/exec',
        params={'query': query}).json()
    df = pd.DataFrame(response['dataset'], columns=pd.DataFrame(response['columns'])['name'].values)
    df.set_index('timestamp', inplace=True)
    df.index = pd.to_datetime(df.index).strftime('%Y-%m-%d')
    df = df.iloc[::-1]
    ## indicator
    df['RSI'] = ta.rsi(df['close'])
    
    adx = ta.adx(df['high'], df['low'], df['close'], 14)
    df[adx.columns] = adx
    
    bbands = ta.bbands(df['close'])
    df[bbands.columns] = bbands
    
    df['SMA200'] = ta.sma(df['close'], 200)
    df['SMA50'] = ta.sma(df['close'], 50)
    df['SMA20'] = ta.sma(df['close'], 20)
    
    macd = ta.macd(df['close'])
    df[macd.columns] = macd
    return df.dropna()

## Tool lấy giá và tính các indicator
ta_tool = Tool(
    name = "Price and technical indicator tools for Vietnam stock",
    description = "Collect stocks prices adn technical indicator for {company_stock} about a specific company in Vietnam stock market.",
    func= lambda symbol: load_price_and_ta(symbol)
)

## Tool search web
search_tool = SerperDevTool(   
    country="vn",locale="vn",location="Vietnam",
    n_results=20,
)
## Tool scraping
scrape_tool = ScrapeWebsiteTool()
```

Sau đó, chúng ta sẽ tiến hành đi xây dựng các nhân sự. 

## Nhân sự AI
Bài viết sử dụng framework Crewai nhằm xây dựng các agent. Để xây dựng 1 agent, ta cần các thông tin sau:
- Role: vai trò của agent này là gì?
- Goal: mục tiêu của agent này là gì?
- Backstory: câu chuyện về agent này là gì?

Bạn có thể đọc thêm tại [Crew.ai](https://blog.crewai.com)


### Technical analyst

```python
## Chuyên viên phân tích kĩ thuật
## Phân tích TA, dự đoán giá trong 20 ngày tới
technicalAnalyst = Agent(
    role= "Senior Technical Analyst",
    goal="""Provide the most recent technical analysis about trend, momentum
            and stockprice prediction in short timeframe (20 days) for targeted stock""",
    backstory="""You're the best at technical analysis, 
    stock price trend prediction and market risk assessment.
    and you're working for a top security firm in Vietnam.""",
    verbose=True,
    tools=[ta_tool],
    allow_delegation=True,
    llm = llm
)
```

### News analyst

```python
## chuyên viên phân tích thông tin
## tìm catalyst, phân tích các yếu tố rủi ro
newsAnalyst = Agent(
    role= "Senior Market Analyst",
    goal="""Provide the most recent news and market sentiment
    analysis that can highly affect stock price performance.
    Also, you must provide the top catalyst as well as risk factors that can highly affect company business performance.""",
    backstory= """
    You're highly experienced in analyzing the market information 
    and risk assessment that can affect company revenue.
    You are also excel in analyzing market sentiment and 
    always skepticism and consider also the source of the news articles. 
    """,
    verbose=True,
    tools=[search_tool,scrape_tool],
    allow_delegation=True,
    llm=llm,
)
```

### Investment advisor

```python
## chuyên viên tư vấn đầu tư
## Phân tích dựa vào 2 nhân tố giá và thông tin để đưa ra tư vấn đầu tư
investmentAdvisor = Agent(
    role= "Senior Investment Advisor",
    goal="""Impress your customers with full analyses over stocks 
    based on technical, and news insights and complete investment recommendations.
    All the information must come from the technicalAnalyst and newsAnalyst.""",
    backstory= """
    You're the best at providing investment advice and risk management
    for the targeted stock. You're also excel in risk management and 
    always consider the risk factors that can affect the investment performance.
    You are now working for a super important customer you need to impress.
    """,
    verbose=True,
    allow_delegation=True,
    tools=[search_tool, scrape_tool],
    llm=llm,
)
```

### Strategy builder

```python
## chuyên viên xây dựng chiến lược
## chiến lược mua hợp lý tối ưu hoá lợi nhuận
tradingAdvisor = Agent(
    role="Trade Advisor",
    goal="""Develop and test various trading strategies 
    based on the technicalAnalyst and investmentAdvisor's recommendation.
    Finally, you must provide the best trading strategy to maximize profit""",
    backstory="This agent specializes in analyzing the timing, price, "
              "and logistical details of potential trades. By evaluating "
              "these factors, it provides well-founded suggestions for "
              "when and how trades should be executed to maximize "
              "efficiency and adherence to strategy.",
    verbose=True,
    allow_delegation=True,
    tools = [ta_tool],
    llm=llm,
)
```

Ta đã có đủ nhân sự rồi, giờ chúng ta sẽ xây dựng các “nhiệm vụ” để hoàn thành được công việc.

Quy trình sẽ như sau:

1. Phân tích giá cổ phiếu, TA
2. Phân tích thông tin, tìm kiếm catalyst, phân tích các rủi ro có thể có
3. Tổng hợp và đưa ra khuyến nghị
4. Xây dựng chiến lược mua bán hợp lý

## Nhiệm vụ

1. Phân tích giá cổ phiếu, TA

```python
## phân tích TA, kháng cự, hỗ trợ
## tìm hiểu xem cổ phiếu quá mua/bán chưa
getStockPrice = Task(
    description= """
    Conduct a technical analysis of {company_stock} stock price 
    trends and patterns, resistance and support level
    utilizing technical indicators that have provided. 
    Considering whether the stock is overbought or oversold.
    """,
    expected_output = """
    The final report must include a detailed analysis 
    of the stock's price trends, key technical indicators, 
    support-resistance level, and any potential buy/sell signals.
    Make sure to provide a clear and concise summary 
    of the stock's current technical position and any potential price targets or risk levels.
    Make sure to use the most recent data possible.
    Finally, please translate the final output to Vietnamese.
    """,
    agent= technicalAnalyst,
    output_file = "TA_analyses.md"
)
```

2. Phân tích thông tin, tìm kiếm catalyst, phân tích các rủi ro có thể có

```python
## phân tích các thông tin gần nhất
## xem xét coi có sự kiện gì sắp xảy ra không
## tâm lý thị trường đang như thế nào, các analyst đang nghĩ gì
get_news = Task(
    description= """
    Collect and summarize recent news articles, press
    releases, and market analyses related to the {company_stock} stock and its industry.
    Pay special attention to any significant events, market sentiments, and analysts' opinions. 
    Also include upcoming events like earnings and others.""",
    
    expected_output = """
    A summary of the overall market and short summary. 
    Include a sentiment score for targeted stock based on the news, 
    notable shifts in market sentiment, and potential impacts for the stock.
    Make sure to use the most recent data as possible.
    Finally, please translate the final output to Vietnamese.
    """,
    agent= newsAnalyst,
    output_file = "news_analyses.md"
)
```

3. Tổng hợp và đưa ra khuyến nghị

```python
## tổng hợp thông tin và đưa ra quyết định có nên tư vấn không
writeAnalyses = Task(
    description = """Use the stock price analysis and the news analysis 
    to create an report and investment advice about the {company_stock} company.
    Focus on the stock price trend, news and market sentiment. 
    What are the near future considerations?
    Include the previous analyses of stock trend and news summary.
    Be straight foward and cite or provide reasons for each of your point.""",
    
    expected_output= """
    A research report of the {company_stock} formated as markdown in an easy readable manner. It should contain:
    - An overall summary
    - A detailed analysis of the stock price trend from the Technical Analyst
    - A detailed analysis of the stock news from the News Analyst
    - Key catalysts and risks for the stock
    - Price prediction for the stock in future and does it over or under value 
    with the current price based on the catalysts and risks
    - A conclusion about key facts and concrete future trend prediction - up, down or sideways. 
    Will it worth to buy or sell the stock when take into account the risk and catalysts.
    Finally, please translate the final output to Vietnamese.
    """,
    agent = investmentAdvisor,
    output_file = "final_report.md"
)
```

4. Xây dựng chiến lược mua bán hợp lý

```python
## xây dựng chiến lược mua khi cổ phiếu quá bán 
## bán khi cổ phiếu có sentiment lên quá cao
tradingStrategy = Task(
    description=
        """Develop and refine trading strategies 
        based on the insights from the 
        technical analyst and investment advisor.
        The objective strategy is buy when the stock is oversold 
        and sell when the sentiment is too high.
        Also, consider the risk factors that can affect the stock price based on the news analysis
        """,
    expected_output=
        """A set of potential trading strategies for {company_stock} 
        that align with the objective startegy.
        Finally, please translate the final output to Vietnamese.""",
    agent=tradingAdvisor,
    output_file = "trading_strategy.md"
)
```

## Kết quả

Mình có lưu lại kết quả ở các phase, các bạn có thể vô đọc file markdown. Ngoài ra, file Full chain of thought sẽ cho ta thấy được toàn thể quá trình vận hành của team tư vấn này ^^.

1. [Phân tích kĩ thuật](https://drive.google.com/open?id=1CwhmWb4BzzaGdX1Iy0V5PP4A5Qj6JOvb&usp=drive_fs)

2. [Phân tích thông tin](https://drive.google.com/open?id=1CqpD187iCnTgWa1ZxOMPFLxausQst2pC&usp=drive_fs)

3. [Tư vấn đầu tư](https://drive.google.com/open?id=1CvPvAF6_YakItsA_ipZ9dxJvZc8UIKH6&usp=drive_fs)

4. [Tư vấn chiến lược giao dịch](https://drive.google.com/open?id=1CcbAi49Uq5J932kQis8plPhc4xV8l1Mt&usp=drive_fs)

5. [Full Chain-of-thought](https://drive.google.com/open?id=1CttFKtbhli6C0CzWSndxTvg3rolIXfmn&usp=drive_fs)

Mình lấy ra 1 vài mẫu mình thấy đặc biệt để cùng nhau cảm nhận (mình làm mẫu trên cổ phiếu HPG)

TA Analyst

```
5. **Tóm tắt vị thế kỹ thuật hiện tại:**
   - Cổ phiếu HPG hiện đang có xu hướng giảm nhưng không bị quá bán. 
   Tuy nhiên, các chỉ số cho thấy có thể có sự phục hồi nhẹ nếu vượt qua ngưỡng kháng cự.
   - Cần theo dõi các mức hỗ trợ và kháng cự để đưa ra quyết định đầu tư hợp lý.

6. **Mục tiêu giá và mức rủi ro:**
   - **Mục tiêu giá ngắn hạn:** Nếu cổ phiếu vượt qua mức kháng cự 27.00 VNĐ, 
   có thể hướng tới mục tiêu 28.00 VNĐ.
   - **Mức rủi ro:** Cần cẩn trọng nếu giá xuống dưới 26.30 VNĐ, 
   có thể dẫn đến sự sụt giảm sâu hơn.

Tóm lại, cổ phiếu HPG hiện tại đang trong giai đoạn điều chỉnh, và cần lưu ý các yếu tố kỹ thuật trước khi tham gia giao dịch.

```

News analyst

```
5. **Sự kiện sắp tới**: 
   - HPG dự kiến sẽ công bố báo cáo lợi nhuận từ ngày 28 tháng 1 đến ngày 3 tháng 2 năm 2025. 
   Đây sẽ là một sự kiện quan trọng, vì phản ứng của thị trường có thể bị ảnh hưởng bởi kết quả.

9. **Yếu tố kích thích và rủi ro**:
   - **Yếu tố kích thích hàng đầu**: 
     - Bứt phá trên mức kháng cự 27,00 VNĐ.
     - Báo cáo lợi nhuận tích cực vào đầu tháng Hai.
   - **Yếu tố rủi ro**:
     - Tiếp tục giảm giá thép.
     - Tăng áp lực bán từ nước ngoài.

```

Advisor

```
2. **Khuyến Nghị Từ Các Nhà Phân Tích**:
   - Các nhà phân tích đồng thuận rằng giá cổ phiếu sẽ tăng 25.5% trong năm tới.
   - Các mục tiêu giá đã được điều chỉnh tăng lên nhiều lần, với mục tiêu mới nhất được đặt là ₫34,075.

3. **Cảm Nhận Thị Trường**:
   - Sự giảm giá gần đây của cổ phiếu đã khiến HPG được xem là không được định giá đúng, làm tăng sự quan tâm của các nhà đầu tư.

## Các Yếu Tố Thúc Đẩy
- **Nhu Cầu Tăng**: Ngành xây dựng tại Việt Nam tiếp tục phát triển, thúc đẩy nhu cầu cho các sản phẩm thép.
- **Tăng Trưởng Lợi Nhuận**: Dự báo tăng trưởng lợi nhuận 24.27% mỗi năm, được thúc đẩy bởi khối lượng bán hàng cao hơn và biên lợi nhuận cải thiện.
- **Vị Trí Thị Trường**: HPG nắm giữ thị phần lớn trong thép xây dựng và đang ở vị trí tốt để tận dụng các xu hướng ngành.

```

Strategy recommendation

```
1. **Tín hiệu Mua**:
   - Khi chỉ số sức mạnh tương đối (RSI) dưới 30, cho thấy cổ phiếu bị bán quá mức. 
   Việc cổ phiếu được phân loại là bị bán quá mức cho thấy rằng nó có thể 
   được định giá thấp, tạo ra cơ hội mua vào tiềm năng.
   - Thêm vào đó, xem xét giá cổ phiếu hiện tại là ₫26,750, 
   thấp hơn khoảng 30.7% so với giá trị hợp lý ước tính, 
   càng củng cố thêm cho trường hợp vào lệnh khi 
   RSI chỉ ra điều kiện đánh giá thấp.

```

## Kết luận

Topic về AI Agent mình tin sẽ sắp được ứng dụng rộng rãi vào thị trường trong 5 năm tới. Việc tạo ra một agent rất dễ. Cái khó ở đây là tạo ra 1 agent đáng tin và đủ giỏi để thực hiện các việc tự động hoá suy luận cao cấp hơn.

Tại start up của mình và các ae (miquant.vn), AI Agent cũng là một trong những sản phẩm đang được chú trọng xây dựng. Thay vì chỉ sử dụng các phương pháp prompt đơn giản như này, tụi mình nâng cấp lên thông qua các công cụ phân tích định lượng cũng như các dự báo kinh tế, expertise include, … nhằm nâng cao tính “trustworthy” của các agent này.

Stay tuned.

## Ref
[Enhancing Investment Analysis: Optimizing AI-Agent Collaboration in Financial Research
](https://arxiv.org/abs/2411.04788)