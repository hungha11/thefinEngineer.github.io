---
title: "Tối ưu hoá danh mục thông qua LLM"
description: "Sử dụng LLM để tối ưu danh mục đầu tư"
date: 2025-02-03
draft: false
math: katex
summary: "Trong kỷ nguyên mới, dữ liệu phi cấu trúc như tin tức, báo cáo phân tích, và thống kê kinh tế trở thành nguồn thông tin quan trọng trong quyết định đầu tư. Tuy nhiên, các mô hình tài chính truyền thống như Lý thuyết Danh mục Hiện đại (MPT) và Tối ưu Rủi ro-Lợi nhuận (Mean-Variance Optimization) chủ yếu dựa vào dữ liệu định lượng, bỏ qua yếu tố định tính quan trọng. Sự phát triển của Large Language Model (LLM) mang đến tiềm năng kết hợp dữ liệu định lượng và dữ liệu phi cấu trúc trong quản lý danh mục đầu tư. Bài viết này trình bày một thực nghiệm ứng dụng LLM trong quá trình tối ưu danh mục."
---

![1.png](images/1.png#center)

Bài viết này thử nghiệm tối ưu hoá danh mục sử dụng Large Language Model (LLM). Thư viện langchain và OpenAI gpt4o và 4o-mini sẽ được sử dụng cho thử nghiệm.

## Concept

### Tối ưu hoá danh mục

Tối ưu hóa danh mục đầu tư là quá trình phân bổ tài sản vào các khoản đầu tư khác nhau nhằm đạt được sự cân bằng tối ưu giữa lợi nhuận kỳ vọng và rủi ro. Các phương pháp tối ưu hóa truyền thống, như Modern Portfolio Theory hay Mean-variance optimization, chủ yếu dựa vào dữ liệu lịch sử và các giả định về phân phối lợi nhuận. Tuy nhiên, chúng có một số hạn chế:

- **Khó khăn trong việc tích hợp các yếu tố định tính**: Các phương pháp này thường không xem xét các yếu tố định tính như tin tức thị trường, sự kiện kinh tế hoặc quan điểm của chuyên gia, tiềm năng từ câu chuyện kinh doanh dẫn đến việc thiếu linh hoạt trong phản ứng với các biến động thị trường.
- **Giả định đơn giản hóa**: Việc giả định rằng lợi nhuận tuân theo phân phối chuẩn và mối quan hệ giữa các tài sản là tuyến tính có thể không phản ánh chính xác thực tế phức tạp của thị trường.

### LLM là gì?

Mô hình ngôn ngữ lớn (LLM) là các mô hình học sâu được huấn luyện trên khối lượng dữ liệu văn bản khổng lồ, giúp chúng có khả năng hiểu và tạo ra ngôn ngữ tự nhiên. Với khả năng đọc hiểu ngôn ngữ này, LLM có thể trích xuất thông tin và phân tích dữ liệu phi cấu trúc nhanh chóng và đơn giản hơn nhiều so với các phương pháp truyền thống. Ví dụ, LLM có thể đọc 10 bài báo gần đây và tổng hợp ra 3 luận điểm đầu tư phổ biến nhất từ các bài báo đó—một nhiệm vụ mà các phương pháp truyền thống khó thực hiện được. 

Tản mạn: một cách khoa học mà nói, chúng ta có thể "hình dung" LLM như sau: LLM là một cỗ máy đưa ra kết quả dựa trên trường hợp có xác suất cao nhất. Các câu prompt đóng vai trò giới hạn phạm vi lựa chọn của LLM, từ đó tăng xác suất cho kết quả mong muốn.

### LLM cho tối ưu hoá danh mục

Với khả năng phân tích và tận dụng các dữ liệu phi cấu trúc trên, LLM là một ứng cử viên sáng giá cho quá trình tối ưu hoá danh mục khi có thể tận dụng được: (1) dữ liệu có cấu trúc, thống kê lợi nhuận rủi ro; (2) dữ liệu phi cấu trúc như tiềm năng tăng trưởng công ty, …; (3) đáp ứng được các yêu cầu về mục tiêu đầu tư (objective), hay các rằng buộc đi kèm (constraints) một các nhanh chóng.

Để dễ dàng hình dung, chúng ta sẽ cùng nhau đi qua thử nghiệm thực tế cho 6 cổ phiếu FPT, HPG, MWG, REE, VCB và VNM!

## Xây dựng danh mục đầu tư sử dụng LLM

Quy trình tối ưu sẽ nhau sau

- B1: Xác định rổ cổ phiếu đầu tư
- B2: Thu thập thông tin: (1) Sentiment của cổ phiếu; (2) Các luận điểm đầu tư - rủi ro của cổ phiếu; (3) tín hiệu kĩ thuật; (4) Độ tương quan giữa các cổ phiếu với nhau
- B3: Tối ưu hoá danh mục: (1) Các mục tiêu đầu tư cũng như giới hạn cho phép; (2) Xem xét danh mục quá khứ và hiệu suất quý gần nhất.
- B4: Backtest

![2.png](images/2.png#center)

### Thu thập tin tức từ google

Tại bước đầu tiên, chúng ta cần xây dựng một pipeline để thu thập được các tin tức liên quan đến từng cổ phiếu, và phải phù hợp với thời gian của giai đoạn backtest. Tại khâu này, chúng ta sẽ sử dụng thư viện langchain để xây dựng pipeline và mô hình gpt-4o-mini để xử lý tin tức

```python
def load_google_news_before(query: str, before_date: datetime.date, num_results: int = 30):
    """
    Query Google News via LangChain’s Google Serper API Wrapper, filtering articles
    so that only those published before the given date are returned.
    """
    cd_min = convert_date_to_str(before_date - datetime.timedelta(days=90))
    cd_max = convert_date_to_str(before_date)
    tbs = f"cdr:1,cd_min:{cd_min},cd_max:{cd_max}"
    search = GoogleSerperAPIWrapper(type="news", tbs=tbs, gl="vn",hl='vi',k=num_results)
    results = search.results(query)
    return results
```

Ta có thể dễ dàng tạo ra một hàm đọc tin tức sử dụng [Serper](https://serper.dev) và thư viện langchain. Hàm trên thiết kế để lấy các tin tức trong vào 90 ngày trước ngày truyền vào.

### Trích xuất các thông tin mong muốn

Để thu thập thông tin phục vụ đầu tư, ta tìm kiếm các bài báo với từ khóa "đầu tư vào cổ phiếu ABC", trong đó ABC là mã cổ phiếu cần phân tích.

Sau khi có được các bài báo liên quan, ta thu thập các thông tin quan trọng bao gồm: (1) sentiment của cổ phiếu trong quý vừa qua; (2) lý giải cho kết quả sentiment đó; (3) các luận điểm đầu tư (Catalyst); (4) các yếu tố rủi ro.

Để thực hiện việc này, ta thiết kế câu prompt dựa trên các yêu cầu trên và sử dụng mô hình LLM gpt-4o-mini. Vì nhiệm vụ này chủ yếu là tổng hợp thông tin, không đòi hỏi nhiều suy luận phức tạp, nên việc sử dụng một mô hình nhỏ gọn, nhanh và tiết kiệm như gpt-4o-mini là lựa chọn phù hợp.

```python
## Sử dụng pydantic để ép model trả về dạng dữ liệu như mong muốn
class Sentiment(BaseModel):
    sentiment: str = Field(default=None, description="The financial market sentiment of all news", enum=["bearish", "bullish", "neutral"])
    explanation: str = Field(default=None, description="The short explanation about the chosen sentiment ")
    catalyst: List[str] = Field(default=None, description="The catalyst based on all news")
    risk_factor: List[str] = Field(default=None, description="The risk factor based on all news")

def extract_key_info(stock , date):
    """
    Extract key information from a news article.
    """
    nest_asyncio.apply()
    query = f"đầu tư cổ phiếu {stock}"
    # For example, retrieve articles published before January 1, 2025.
    cutoff_date = date
    articles = load_google_news_before(query, cutoff_date, num_results=15)
    links = pd.DataFrame(articles['news'])['link'].to_list()
    ## drop the link from https://etime.danviet.vn
    links = [link for link in links if 'danviet.vn' not in link] ## skip for danviet.vn as it block the request
    loader = WebBaseLoader(links, continue_on_failure=True)

    loader.requests_per_second = 0.5
    docs = loader.aload()
    ls = []
    for doc in docs:
        ls.append(doc.page_content.replace('\n\n', '').replace('\n', ' '))

    llm = ChatOpenAI(model='gpt-4o-mini')

    prompt = """
    You're a senior analyst. You are analyzing stock {stock}
    Your tasks are: 
    - Analyze the sentiment of the given news
    - Explain the sentiment in few words
    - Identify the key catalysts or drivers
    - Identify the risk factors

    {document}

    Requirement
    - Be specific and concise
    - Do not make up information or make assumption
    - Do not use external source
    """.format(stock = stock,document=ls)

    response = llm.with_structured_output(Sentiment).invoke(prompt)
    return response
    
    
date = datetime.date(2024, 1, 1)
extract_key_info('HPG', date)
```

Kết quả của hàm trên sẽ như sau:

Sentiment(sentiment='bearish', explanation='Sentiment is bearish due to significant losses reported by HPG and related companies, along with ongoing challenges in the steel market.', catalyst=['Increased production and sales in October 2023', 'Expected recovery in demand and prices for steel', 'Completion of Dung Quất 2 project may increase capacity'], risk_factor=['High competition in the steel industry', 'Dependence on domestic market recovery', 'Fluctuations in raw material prices'])

### Tối ưu hoá danh mục

Khi tối ưu hoá danh mục, việc xây dựng câu prompt chi tiết và cụ thể sẽ giúp giảm thiểu lỗi và tạo ra tỷ trọng hợp lý hơn. Do độ phức tạp tại khâu này tăng cao, chúng ta cần sử dụng một mô hình LLM mạnh mẽ hơn. Bài viết chọn sử dụng **gpt-4o.**

**Mục tiêu đầu tư**: Phân bổ tỷ trọng cho các cổ phiếu có tiềm năng tăng trưởng cao, dựa trên tín hiệu kỹ thuật, sentiment và các luận điểm đầu tư.

**Các rằng buộc:**

- Long-only portfolio
- Không sử dụng margin
- Hold trong 3 tháng
- Không cần đầu tư hết 6 cổ phiếu nhưng phải có ít nhất 4 cổ phiếu

Ngoài ra, câu prompt cũng bao gồm một workflow (quy trình) để LLM có thể dựa vào đó để đưa ra quyết định phân bổ tỷ trọng

1. Dựa trên các luận điểm đầu tư để xác định các cổ phiếu tiềm năng
2. Đánh giá sentiment của các cổ phiếu đã chọn
3. Sử dụng tín hiệu kỹ thuật để xác nhận thêm tiềm năng của các cổ phiếu
4. Cuối cùng, sử dụng dữ liệu rủi ro-lợi nhuận của 3 tháng qua để xác định các cổ phiếu được mua quá mức hoặc bị thổi phồng, tránh phân bổ quá nhiều vốn cho chúng. Tuy nhiên, nếu luận điểm đầu tư hứa hẹn, ta vẫn có thể phân bổ tiền cho cổ phiếu đó nếu tin rằng nó có tiềm năng đáng kể
5. Chia tỷ trọng cho từng cổ phiếu dựa trên tiềm năng của các luận điểm đầu tư
6. Phân bổ nhiều tỷ trọng hơn cho các cổ phiếu có luận điểm đầu tư khả thi và tiềm năng cao

```python
prompt = """
Create a portfolio with the following stocks: {stocks}

Objective:
- Use the technical signal, market sentiment and catalysts to assign weight to each stock for a long-only portfolio
- The portfolio aim to invest in the stock with the highest potential return

Constraints:
- The portfolio should be well diversified
- You can invest in each stock with a maximum of 0.5 of the portfolio
- You do not need to invest in all stocks, but you must invest in at least 4 stocks. Only select the most promising stocks.
- There is no leverage & short selling
- The holding period is 3 months (1 quarter)
- The weight should be a float number between 0 and 1
- The sum of all weights should be 1

Workflow:
1. Analyze information first. 
2. Based on the catalysts, identify the stocks that are likely to perform well.
3. Assess the sentiment of the news to confirm the potential of the selected stocks.
4. Use the technical signal to further confirm the potential of the stocks.
5. Finally, use the past 3 months' risk-return data to identify overbought or hyped stocks to avoid allocating too much capital to them. However, if the catalyst is highly promising, you may still allocate funds to it if you believe it has significant potential.
6. Assign weights to each stock based on the potential return driven by the catalyst.
7. Allocate more weight to stocks where the catalyst is more likely to occur and has a higher magnitude.
8. Be confident in assigning higher weights to stocks with better catalysts and sentiment.
9. **Think carefully; do not simply split the weights equally.**

Technical signal: simple EMA50-200 crossover, 1 for bullish, 0 for bearish
{ta_signal}

Past 3 risk return
{risk_return}

News sentiment (0 for bearish, 1 for bullish, 2 for neutral) & Catalysts
{sentiment}

Correlation between stocks
{correlation}

Your previous portfolio
{past_portfolios}

Your previous return: {last_period_return}

Output
- A list with weight for every stock. The weight must correspond to the oder {stocks}. For stock that not invest, simple return 0 for the weight
- A detailed explanation of the rationale behind the portfolio allocation
"""
```

Sau đó, ta sẽ tính toán các tham số và đưa các thông tin cần thiết vào trong mô hình để mô hình thực hiện và tính toán.

```python
class PortfolioOptimizationResult(BaseModel):
    """
    A class to represent the result of portfolio optimization.
    """
    stock_weight: List[float] = Field(
        default=None,
        description="A list containing the weight for all stocks in the portfolio."
    )
    explanation: str = Field(
        default=None,
        description="A detailed explanation of the rationale behind the portfolio allocation."
    )
    
def llm_optimizer(date, stocks = ['FPT', 'HPG', 'MWG', 'REE', 'VCB', 'VNM'], past_portfolios = None, last_period_return = None):
    results = run_in_parallel(stocks, date)
    
    llm = ChatOpenAI(model='gpt-4o')
    ta_signal = ta_compare[:date].iloc[-1]
    
    start_sample = date - datetime.timedelta(days=30*3) ## 3 months
    sample_rets = rets[start_sample:date]
    risk_return = ((1+sample_rets).product()-1).to_frame()
    risk_return.columns = ['past_3m_rets']
    risk_return['past_3m_volatility'] = sample_rets.std()
    risk_return['past_3m_sharpe_ratio'] = sample_rets.mean()/risk_return['past_3m_volatility']
    risk_return = risk_return.round(4)
    
    correlation = sample_rets.corr().round(4)   
    
    chatprompt = prompt.format(ta_signal=ta_signal, sentiment=results, risk_return=risk_return, stocks=stocks, correlation=correlation, past_portfolios=past_portfolios, last_period_return=last_period_return)
    optimizer = llm.with_structured_output(PortfolioOptimizationResult).invoke(chatprompt)
    
    weight = pd.DataFrame(optimizer.stock_weight, index=stocks, columns=['weights'])
    weight = weight/weight.sum() ## make sure the sum of weight is 1
    retionale = optimizer.explanation
    return weight, retionale
    
weight, expla = llm_optimizer(datetime.date(2024, 6, 1)) ## this takes around 35 seconds
```

Sau khi chạy hàm trên, ta sẽ nhận được kết quả như sau (danh mục cho tháng 6.2024):

Tỷ trọng danh mục

|  | weights |
| --- | --- |
| FPT | 0.25 |
| HPG | 0.2 |
| MWG | 0.3 |
| REE | 0.15 |
| VCB | 0.1 |
| VNM | 0.0 |

Lý do cho tỷ trọng trên

<aside>
💡

In constructing this portfolio, we focused on stocks with strong bullish technical signals and favorable market sentiment, while considering catalysts and risks.

1. **FPT (Weight: 0.25)**: FPT has a bullish technical signal, strong sentiment driven by its partnership with NVIDIA for AI development, and projected revenue growth. Although its valuation is high, its catalysts are promising, warranting a significant allocation.
2. **HPG (Weight: 0.2)**: HPG also exhibits a bullish technical signal and optimistic market sentiment due to strong Q1 financial results and upcoming projects like Dung Quất 2. Despite high debt levels, its growth potential justifies a substantial allocation.
3. **MWG (Weight: 0.3)**: MWG has demonstrated strong past returns and a bullish sentiment, supported by strategic investments and projected growth. The high weight reflects its potential for recovery and profit, despite some risks.
4. **REE (Weight: 0.15)**: REE's neutral sentiment and mixed outlook are balanced by its dividend plans and growth in renewable energy. Its lower weight reflects these uncertainties, but its diversification benefits and potential in energy justify inclusion.
5. **VCB (Weight: 0.1)**: Despite a neutral sentiment and recent negative returns, VCB's catalysts such as planned capital increases and potential equity sales offer upside potential, meriting a smaller allocation.
6. **VNM (Weight: 0)**: VNM is excluded due to bearish sentiment and negative past performance, with significant foreign sell-offs and declining market position, making it less attractive for the portfolio.

Overall, this portfolio is designed to be diversified and capitalize on stocks with strong potential returns driven by clear catalysts, while mitigating risks associated with each stock's market position and sentiment.

</aside>

### Backtest cho phương pháp trên

Ta tiến hành xây dựng mô hình backtest đơn giản.

- Mua danh mục vào đầu mỗi quý
- Bỏ qua các yếu tố như phí giao dịch, market impact

```python
## backtest the strategy - >=35s for 1 loop -> 4 years = 4*4*35s >= 560s ~ 9.3 minutes
## create a function that takes the weights and returns the portfolio returns
def backtest_strategy(weights, returns):
    """
    Backtest the strategy
    """
    port_rets =  returns @ weights
    port_rets.rename(columns={'weights':'rets'}, inplace=True)
    return port_rets

date_idx = pd.date_range('2020-12-31', '2024-12-31', freq='QE')

llm_rets = pd.DataFrame()
llm_retionale = {}
llm_weights = {}
for date in date_idx[:-1]:
    i = 0
    start_sample = date - pd.DateOffset(years=3)
    end_sample = date
    start_out_sample = date+ pd.DateOffset(days=1)
    end_out_sample = date + pd.DateOffset(months=3)
    
    ## data for the sample period
    sample_data = rets[start_sample:end_sample]
    out_sample = rets[start_out_sample:end_out_sample]
    
    # run the optimizer
    if i >=1:
        past_port = pd.DataFrame(llm_weights[previous_date.strftime("%Y-%m-%d")])
        w, explanation = llm_optimizer(date, past_portfolios=past_port, last_period_return= past_return)
    else:
        w, explanation = llm_optimizer(date)
        
    llm_retionale[date.strftime("%Y-%m-%d")] = explanation
    llm_weights[date.strftime("%Y-%m-%d")] = w.to_dict()["weights"]
    
    # run the backtest
    llm_backtest = backtest_strategy(w, out_sample)
    llm_rets = pd.concat([llm_rets, llm_backtest], axis=0)
    
    i+=1
    previous_date = date
    past_return = (1+llm_backtest).product()-1
```

Từ đó, ta thu được kết quả sau:

Lợi nhuận danh mục từ 2021 tới nay

![3.png](images/3.png#center)

Thống kê liên quan

|  | **LLM** | **VNINDEX** |
| --- | --- | --- |
| **Lợi nhuận tích luỹ** | 70.9% | 12.6% |
| **Lợi nhuận trung bình hàng năm** | 14.52% | 3.04% |
| **Độ biến động trung bình hằng năm** | 21.66% | 19.6% |
| **Sharpe ratio** | 0.73 | 0.25 |
| **Lần sụt giảm mạnh nhất** | 24.19% | 40.34% |

Chiến lược sử dụng LLM cho kết quả tương đối tốt với lợi nhuận tích luỹ 70.9% so với 12.6% của vnindex qua 4 năm. Tuy nhiên, tỷ lệ sharpe thì tương đối khiêm tốn chỉ 0.73 với độ biến động 21.6%. Ngoài ra lần sụt giảm lớn nhất đạt 24.19% là một kết quả chấp nhận được trong đầu tư.

Danh mục qua từng giai đoạn

![4.png](images/4.png#center)

## Kết luận

Bài viết đã tiến hành xây dựng tối ưu hoá danh mục sử dụng LLM. Thông qua LLM, ta có thể đưa các dữ liệu phi cấu trúc (unstructured-data) vào trong quá trình tối ưu. Đây là một điểm thú vị và các phương pháp truyền thống tận dụng các tính chất thống kê chưa đưa vào được. Tuy nhiên, phải nhấn mạnh lại rằng, tối ưu danh mục theo phương pháp này thì mang nặng yếu tố định tính (qualitative) hơn so với định lượng (quantitative). 

Một điểm thú vị này, ta có thể kết hợp 2 phương pháp này với nhau. Thông qua LLM, ta có thể tìm ra một prior belief mang tính định tính và dùng nó để bắt đầu quá trình tối ưu hoá định lượng hơn (bayesian style).

Bài viết sau sẽ tiến hành so sánh các phương pháp truyền thống với các phương pháp mới này. Để nhận file code và data đầy đủ, xin hãy để comment vào bài viết trên linkedin hoặc gửi email đến hung.ha@miquant.vn.