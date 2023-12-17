# Optimization

## Battery Optimization
- Batching request
  - If in one page a lot of calling API, consider call in batch
    - pros: battery and network optimization.
    - cons: If 1 api error the page couldn't be loaded.
- Websocket lifecycle awareness
  - If app goes background websocket is still collecting and pending in flow if app start again flow will start collect and update UI, main problem with this is battery consumption still drain and network still consume. 
    - We can make websocket lifecycle awareness when app in pause we can pause collecting websocket and try to reconnect after onstart, maybe we can start batch from be like <code>pull $unixtime</code> unixtime is time that app goes pause, that means BE please return all data after unixtime

## Slow Network Optimization
- Load image low res first
  - If most users network is slow, consider load low image first and make image size as view size
    - pros: network slow experience better
    - cons: take higher network consumption