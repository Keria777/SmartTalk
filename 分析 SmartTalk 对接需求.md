# 分析 SmartTalk 对接需求

> **Workspace:** /Users/keria/AIProjectWorkSpace/SMT/SmartTalk

---

## User
有新需求。描述如下： 
 
 Story ：As a 銷售人員，I want to Smartalk AI 对接銷售CRM系统的客戶跟進類型，So that 識別客人ID所屬公司，以便查詢對應公司的報價。 
 
 As a 銷售人員，I want to Smartalk AI 对接Hifood系统查询商品的價格，So that 客人询问"商品價格"时，能获知其所咨詢商品的價格。 
 
 Scenario： 
 
 SJ客户查询商品价格 
 
 1.客人（SJ客户）来电。 
 
 2.AI匹配到该号码对应唯一ID，CRM判定其客户类型为“SJ”。 
 
 3.客人咨询商品B的价格。 
 
 4.系统在Hifood中匹配到商品B，并查询其“SJ AI报价”字段。 
 
 5.AI回答价格信息。 
 
 OS客户查询商品价格 
 
 1.客人（OS客户）来电。 
 
 2.AI匹配到该号码对应唯一ID，CRM判定其客户类型为“OS”。 
 
 3.客人咨询商品B的价格。 
 
 4.系统在Hifood中匹配到商品B，并查询其“子公司 KF/OS AI报价”字段。 
 
 5.AI回答价格信息。 
 
 KF客户查询商品价格 
 
 1.客人（KF客户）来电。 
 
 2.AI匹配到该号码对应唯一ID，CRM判定其客户类型为“KF”。 
 
 3.客人咨询商品B的价格。 
 
 4.系统在Hifood中匹配到商品B，并查询其“子公司 KF/OS AI报价”字段。 
 
 5.AI回答价格信息。 
 
 该需求关联3个项目，分别是这个SmartTalk和公司的HiFood1.0和CRM2.0。 我第一次上手SmartTalk这个项目，可以帮我分析一下这个需求吗？这个需求是需要做什么？在这个项目里要如何做？

## Assistant
首先需要了解SmartTalk项目的结构，以便分析如何实现对接CRM和Hifood系统的需求。我将查看项目的文件结构。

现在我需要查看与客户管理和商品价格查询相关的控制器和服务，以了解如何实现与CRM和Hifood系统的对接。

我需要查看与销售相关的代码，特别是Sales目录和服务，以了解如何实现与CRM和Hifood系统的对接。

现在我需要查看与客户类型和商品价格查询相关的服务，特别是如何与CRM和Hifood系统对接的代码。

我需要查看与CRM和Hifood系统对接的相关代码，特别是CrmClient和SalesClient，以了解如何实现客户类型识别和商品价格查询功能。

现在我需要查看SalesClient，以了解如何与Hifood系统对接，以及如何实现商品价格查询功能。

# 需求分析与实现方案

## 需求理解

根据提供的需求描述，您需要SmartTalk AI系统实现以下功能：

1. **对接销售CRM系统**：
   - 识别客人ID所属公司
   - 判定客户类型（SJ、OS、KF）
   - 以便查询对应公司的报价

2. **对接Hifood系统**：
   - 查询商品的价格
   - 根据不同客户类型查询不同的价格字段：
     - SJ客户：查询"SJ AI报价"字段
     - OS客户：查询"子公司 KF/OS AI报价"字段
     - KF客户：查询"子公司 KF/OS AI报价"字段

3. **场景流程**：
   - 客人来电 → AI匹配号码对应唯一ID → CRM判定客户类型 → 客人咨询商品价格 → 系统在Hifood中匹配商品并查询对应价格字段 → AI回答价格信息

## 现有代码分析

通过查看SmartTalk项目的代码，我发现：

1. **已有CRM对接能力**：
   - `CrmClient` 类已实现与CRM系统的基本交互，包括获取客户信息和联系信息
   - 已有 `GetCustomersByPhoneNumberAsync` 方法可以通过电话号码获取客户信息

2. **已有Sales系统对接能力**：
   - `SalesClient` 类已实现与销售系统的交互，包括获取订单历史、客户习惯等
   - 但目前缺少查询商品价格的方法

3. **AI对话处理能力**：
   - 项目中已有 `AiSpeechAssistantService` 和 `AiSpeechAssistantConnectService` 等服务处理AI对话
   - 已有功能调用相关代码，可扩展添加商品价格查询功能

## 实现方案

### 1. 扩展CRM客户端

首先，需要扩展 `CrmClient` 以支持获取客户类型：

```csharp /Users/keria/AIProjectWorkSpace/SMT/SmartTalk/src/SmartTalk.Core/Services/Http/Clients/CrmClient.cs
public interface ICrmClient : IScopedDependency
{
    // 现有方法...
    
    // 添加新方法：获取客户类型
    Task<string> GetCustomerTypeAsync(string customerId, CancellationToken cancellationToken);
}

public class CrmClient : ICrmClient
{
    // 现有代码...
    
    public async Task<string> GetCustomerTypeAsync(string customerId, CancellationToken cancellationToken)
    {
        var token = await GetCrmTokenAsync(cancellationToken).ConfigureAwait(false);
        
        var headers = new Dictionary<string, string>
        {
            { "Accept", "application/json" },
            { "Authorization", $"Bearer {token}" }
        };
        
        // 假设CRM系统提供了获取客户类型的API
        var url = $"{_crmSetting.BaseUrl}/api/customer/{customerId}/type";
        
        return await _httpClient
            .GetAsync<string>(url, headers: headers, cancellationToken: cancellationToken)
            .ConfigureAwait(false);
    }
}
```

### 2. 扩展Sales客户端（Hifood对接）

需要扩展 `SalesClient` 以支持查询商品价格：

```csharp /Users/keria/AIProjectWorkSpace/SMT/SmartTalk/src/SmartTalk.Core/Services/Http/Clients/SalesClient.cs
public interface ISalesClient : IScopedDependency
{
    // 现有方法...
    
    // 添加新方法：查询商品价格
    Task<GetProductPriceResponseDto> GetProductPriceAsync(GetProductPriceRequestDto request, CancellationToken cancellationToken);
}

public class SalesClient : ISalesClient
{
    // 现有代码...
    
    public async Task<GetProductPriceResponseDto> GetProductPriceAsync(GetProductPriceRequestDto request, CancellationToken cancellationToken)
    {
        // 假设Hifood系统提供了查询商品价格的API
        var url = $"{_salesSetting.BaseUrl}/api/Product/GetPrice";
        
        return await _httpClientFactory.PostAsJsonAsync<GetProductPriceResponseDto>(url, request, headers: _headers, cancellationToken: cancellationToken).ConfigureAwait(false);
    }
}
```

### 3. 添加DTO类

需要添加相关的DTO类：

```csharp /Users/keria/AIProjectWorkSpace/SMT/SmartTalk/src/SmartTalk.Messages/Dto/Sales/GetProductPriceRequestDto.cs
namespace SmartTalk.Messages.Dto.Sales;

public class GetProductPriceRequestDto
{
    public string ProductName { get; set; }
    public string CustomerType { get; set; } // SJ, OS, KF
}
```

```csharp /Users/keria/AIProjectWorkSpace/SMT/SmartTalk/src/SmartTalk.Messages/Dto/Sales/GetProductPriceResponseDto.cs
namespace SmartTalk.Messages.Dto.Sales;

public class GetProductPriceResponseDto
{
    public decimal Price { get; set; }
    public string PriceType { get; set; } // SJ AI报价, 子公司 KF/OS AI报价
}
```

### 4. 扩展AI对话处理

需要在AI对话处理服务中添加商品价格查询功能：

```csharp /Users/keria/AIProjectWorkSpace/SMT/SmartTalk/src/SmartTalk.Core/Services/AiSpeechAssistantConnect/AiSpeechAssistantConnectService.FunctionCalls.cs
// 添加商品价格查询功能
public async Task<FunctionCallResult> HandleGetProductPriceAsync(GetProductPriceRequest request, CancellationToken cancellationToken)
{
    // 1. 根据电话号码获取客户信息
    var customerRequest = new GetCustmoersByPhoneNumberRequestDto
    {
        PhoneNumber = request.PhoneNumber
    };
    
    var customers = await _crmClient.GetCustomersByPhoneNumberAsync(customerRequest, cancellationToken).ConfigureAwait(false);
    
    if (customers == null || !customers.Any())
    {
        return new FunctionCallResult
        {
            Success = false,
            Message = "未找到客户信息"
        };
    }
    
    // 2. 获取客户类型
    var customerId = customers.First().CustomerId;
    var customerType = await _crmClient.GetCustomerTypeAsync(customerId, cancellationToken).ConfigureAwait(false);
    
    // 3. 查询商品价格
    var priceRequest = new GetProductPriceRequestDto
    {
        ProductName = request.ProductName,
        CustomerType = customerType
    };
    
    var priceResponse = await _salesClient.GetProductPriceAsync(priceRequest, cancellationToken).ConfigureAwait(false);
    
    return new FunctionCallResult
    {
        Success = true,
        Data = new
        {
            ProductName = request.ProductName,
            Price = priceResponse.Price,
            PriceType = priceResponse.PriceType,
            CustomerType = customerType
        }
    };
}
```

### 5. 配置OpenAI工具

需要在OpenAI工具配置中添加商品价格查询工具：

```csharp /Users/keria/AIProjectWorkSpace/SMT/SmartTalk/src/SmartTalk.Core/Constants/OpenAiToolConstants.cs
// 添加商品价格查询工具
public const string GetProductPriceTool = "get_product_price";
public static readonly OpenAiTool GetProductPriceToolDefinition = new OpenAiTool
{
    Type = "function",
    Function = new OpenAiFunction
    {
        Name = GetProductPriceTool,
        Description = "查询商品价格，根据客户类型返回不同的价格",
        Parameters = new OpenAiFunctionParameters
        {
            Type = "object",
            Properties = new Dictionary<string, OpenAiFunctionProperty>
            {
                {
                    "phoneNumber",
                    new OpenAiFunctionProperty
                    {
                        Type = "string",
                        Description = "客户电话号码"
                    }
                },
                {
                    "productName",
                    new OpenAiFunctionProperty
                    {
                        Type = "string",
                        Description = "商品名称"
                    }
                }
            },
            Required = new[] { "phoneNumber", "productName" }
        }
    }
};
```

## 实现步骤

1. **扩展CRM客户端**：添加获取客户类型的方法
2. **扩展Sales客户端**：添加查询商品价格的方法
3. **添加DTO类**：定义请求和响应的数据结构
4. **扩展AI对话处理**：添加商品价格查询功能
5. **配置OpenAI工具**：添加商品价格查询工具定义
6. **测试验证**：确保功能正常工作

## 技术要点

1. **系统集成**：通过HTTP客户端与CRM和Hifood系统进行集成
2. **客户类型识别**：根据CRM系统返回的客户信息判定客户类型
3. **价格字段选择**：根据客户类型选择不同的价格字段
4. **AI功能调用**：通过OpenAI工具调用实现商品价格查询
5. **错误处理**：处理可能的异常情况，如未找到客户信息、商品价格等

## 注意事项

1. **API接口**：需要确认CRM和Hifood系统提供的具体API接口，可能需要根据实际情况调整代码
2. **认证方式**：确保正确配置API密钥和认证信息
3. **数据安全**：确保客户信息和价格信息的安全传输
4. **性能优化**：考虑添加缓存机制，减少重复API调用
5. **错误处理**：添加适当的错误处理和日志记录

通过以上实现方案，SmartTalk AI系统将能够根据客户类型查询对应公司的商品价格，满足销售人员的需求。

