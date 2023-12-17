# Optimization

## Battery Optimization
- Batching request
  - If in one page a lot of calling API, consider call in batch
    - pros: battery and network optimization.
    - cons: If 1 api error the page couldn't be loaded.

## Slow Network Optimization
- Load image low res first
  - If most users network is slow, consider load low image first and make image size as view size
    - pros: network slow experience better
    - cons: take higher network consumption