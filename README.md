# qr-emvco-decoder-swift

# How to use?

```
let qrEmvData = 00020101021143520016com.mymerchantar0128http://localhost:3344/p/931850150011203611100015204970053030325802AR5921SERGIO MATIAS GUALINO6004CABA63044edf
decodeEMVQR(qr: qrEmvData)

// This function returns 
/*
 MERCHANT_ID_DATA 00 = 20361110001
 COUNTRY_CODE = AR
 MERCHANT_NAME = SERGIO MATIAS GUALINO
 MERCHANT_CITY = CABA
 MERCHANT_URL_DATA 00 = com.mymerchantar
 MERCHANT_URL_DATA 01 = http://localhost:3344/p/9318
 MERCHANT_CATEGORY_CODE = 9700
 CURRENCY_CODE = 032
*/
```


## Define keys 

```
let PAYLOAD_FORMAT_INDICATOR = "00"
let POINT_INITIATION_METHOD = "01"
let MERCHANT_URL_DATA = "43"
let MERCHANT_ID_DATA = "50"
let MERCHANT_CATEGORY_CODE = "52"
let CURRENCY_CODE = "53"
let TRANSACTIONAL_AMOUNT = "54"
let COUNTRY_CODE = "58"
let MERCHANT_NAME = "59"
let MERCHANT_CITY = "60"
let CRC = "63"

let keys = [
    MERCHANT_URL_DATA: "MERCHANT_URL_DATA",
    MERCHANT_ID_DATA: "MERCHANT_ID_DATA",
    MERCHANT_CATEGORY_CODE: "MERCHANT_CATEGORY_CODE",
    CURRENCY_CODE: "CURRENCY_CODE",
    COUNTRY_CODE: "COUNTRY_CODE",
    MERCHANT_NAME: "MERCHANT_NAME",
    MERCHANT_CITY: "MERCHANT_CITY"
]

let emvKeys = [MERCHANT_URL_DATA, MERCHANT_ID_DATA]
```

## Functions

`parseData` recibes TLV (Tag, Length, Value) string and returns the key, the value and the rest of the string.

```
func parseData(data: String) -> (String, String, String)? {
    let keyIndex = data.index(data.startIndex, offsetBy: 1)
    let key = String(data[...keyIndex])
    
    let tempStartIndex = data.index(data.startIndex, offsetBy: 2)
    let temp = String(data[tempStartIndex...])
    
    let lengthIndex = temp.index(temp.startIndex, offsetBy: 1)
    if let length = Int(String(temp[...lengthIndex])) {
        let valueStartIndex = temp.index(temp.startIndex, offsetBy: 2)
        let valueEndIndex = temp.index(temp.startIndex, offsetBy: 1 + length)
        let value = String(temp[valueStartIndex...valueEndIndex])
        
        let newDataIndex = temp.index(temp.startIndex, offsetBy: 2 + length)
        let newData = String(temp[newDataIndex...])
        
        return (key, value, newData)
    }
    
    return nil
}
```

`decodeEMV` recibes a valid QR EMV data (TLV) and returns dictionary formed with the keys and values included in the TLV data.

```
func decodeEMV(qrData: String) -> [String:String]? {
    var data = qrData
    var dic = [String:String]()

    while data.isEmpty == false {
        if let dataParsed = parseData(data: data) {
            data = dataParsed.2
            
            if dataParsed.0.isEmpty == false, dataParsed.1.isEmpty == false {
                dic[dataParsed.0] = dataParsed.1
            }
        }
    }

    return dic
}
```

`decodeEMVQR` prints all parsed data transforming the key to a readable string and parsing a value when is a TLV data.

```
func decodeEMVQR(qr: String) {
    guard let dic = decodeEMV(qrData: qr) else {
        print("Error")
        return
    }
    
    keys.forEach({
        let key = $0.key
        if let value = dic[key] {
            if emvKeys.contains($0.key), let dic = decodeEMV(qrData: value) {
                dic.enumerated().forEach({
                    if let _key = keys[key] {
                        print("\(_key) \($0.element.key) = \($0.element.value)")
                    }
                })
            } else {
                print("\($0.value) = \(value)")
            }
        }
    })
}
```
