`nyaruka/phonenumbers` 仓库地址：https://github.com/nyaruka/phonenumbers

### 基本使用

`nyaruka/phonenumbers` 是 Go 语言中用于处理电话号码的库。它提供了功能强大的工具，用于解析、格式化和验证电话号码，同时还支持国际电话号码的处理。

这个库的主要用途包括：

1. **解析**：能够将电话号码字符串解析成可操作的对象，提取出国家代码、区号、本地号码等信息。
2. **格式化**：能够将电话号码格式化成特定的格式，符合国际或特定国家/地区的电话号码格式。
3. **验证**：可以验证电话号码是否符合特定国家/地区的电话号码规则。

这里有一个简单的例子来展示它的基本用法：

```go
package main

import (
	"fmt"
	"github.com/nyaruka/phonenumbers"
)


func main() {
	number := "+8618389877234"

		// 解析电话号码
	parsedNumber, err := phonenumbers.Parse(number, "CN")
	if err != nil {
		fmt.Println("解析错误：", err)
		return
	}

	// 获取国家代码
	regionCode := phonenumbers.GetRegionCodeForNumber(parsedNumber)
	fmt.Println("国家代码：", regionCode)
	// 国家代码： CN

	countryCode := parsedNumber.GetCountryCode()
	fmt.Println("country code：", countryCode)
	// 86

	// 格式化电话号码
	formattedNumber := phonenumbers.Format(parsedNumber, phonenumbers.NATIONAL)
	fmt.Println("格式化后的号码:", formattedNumber)
	// 格式化后的号码: 183 8987 7234

	// 验证电话号码
	isValid := phonenumbers.IsValidNumber(parsedNumber)
	fmt.Println("是否有效号码:", isValid)
	// 是否有效号码: true
}
```

这个例子演示了如何使用 `nyaruka/phonenumbers` 库来解析、格式化和验证电话号码。你可以根据需要使用这些功能，并根据具体的需求调整代码。

### 格式化电话号码

```go
// 格式化电话号码
	formattedNumber := phonenumbers.Format(parsedNumber, phonenumbers.NATIONAL)
	fmt.Println("格式化后的号码:", formattedNumber)
	// 格式化后的号码: 183 8987 7234
```

 `PhoneNumberFormat` 枚举类型：

- `E164`: E.164 格式，它是国际电话号码的标准格式，包括国家代码、区号和本地号码。例如：+14155552671。
- `INTERNATIONAL`: 国际格式，通常包括国家代码，但不包括国际呼叫前缀（如+号）。例如：4155552671。
- `NATIONAL`: 国内格式，即特定国家或地区的本地格式。例如：(415) 555-2671。
- `RFC3966`: RFC 3966 格式，这是一种用于表示电话号码的另一种国际标准格式。

这些枚举值可以用于指定在处理电话号码时，应该以何种格式来呈现或存储这些号码。例如，你可以使用这些枚举值中的一个来指定将电话号码格式化成特定的形式，以适应不同的需求和标准。

#### FormatByPattern 按指定规则格式化

`FormatByPattern` 函数用于按照特定的模式和客户自定义的格式规则对电话号码进行格式化。

如果电话号码具有零作为国家呼叫代码或者其他无效的国家呼叫代码，那么一些国家特有的格式化规则（比如是否需要应用国家前缀，如何格式化分机号等）可能无法确定，因此在这种情况下，函数会返回未应用任何格式化的国家重要号码（national significant number）。

```go
func FormatByPattern(number *PhoneNumber,
	numberFormat PhoneNumberFormat,
	userDefinedFormats []*NumberFormat) string {
	 ...
	}
```

参数：

- `number *PhoneNumber`：要进行格式化的电话号码对象。
- `numberFormat PhoneNumberFormat`：指定要应用的电话号码格式化类型（E164、INTERNATIONAL、NATIONAL、RFC3966等）。
- `userDefinedFormats []*NumberFormat`：用户定义的格式化规则，可能包含特定模式的自定义格式化方式。

示例：

```go
formatted := strings.SplitN(phonenumbers.FormatByPattern(num, phonenumbers.INTERNATIONAL, []*phonenumbers.NumberFormat{
    {
       Pattern: s(`(\d+)`),
       Format:  s(`$1`),
    },
}), " ", 2)
```



### 判断是否是移动电话号码：

```go
func isMobilePhoneNumber(num *phonenumbers.PhoneNumber) bool {
	numberType := phonenumbers.GetNumberType(num)
	return numberType == phonenumbers.MOBILE || numberType == phonenumbers.FIXED_LINE_OR_MOBILE
}
```

其中电话号码类型`PhoneNumberType`有：

- `FIXED_LINE`: 固定电话号码。
- `MOBILE`: 移动电话号码。
- `FIXED_LINE_OR_MOBILE`: 在某些地区（比如美国），通过电话号码本身无法区分固定电话和移动电话。
- `TOLL_FREE`: 免费电话号码，通常用于客户服务或特定目的。
- `PREMIUM_RATE`: 高费率电话号码，通常收取较高费用。
- `SHARED_COST`: 分摊成本电话号码，呼叫费用由呼叫者和接收者共同承担。
- `VOIP`: 互联网电话号码，即使用 Voice over IP（VoIP）技术的电话号码。
- `PERSONAL_NUMBER`: 个人号码，关联特定个人，可能路由到移动电话或固定电话。
- `PAGER`: 传呼机号码。
- `UAN`: 通用访问号码或公司号码，可以进一步路由到特定办公室，但允许一个号码用于整个公司。
- `VOICEMAIL`: 语音信箱接入号码。
- `UNKNOWN`: 未知类型的电话号码，即不符合任何已知区域的电话号码模式。

这些枚举值帮助在处理电话号码时对其类型进行分类，以便更好地理解和处理不同类型的电话号码。