- **工作模式**：
  QSPI支持多种数据传输模式，具体取决于数据线的数量。常见的传输模式包括：
  1. **单线模式（Single I/O）**：使用一条数据线进行传输，类似于传统的SPI。
  2. **双线模式（Dual I/O）**：使用两条数据线来实现双倍的数据传输速率。
  3. **四线模式（Quad I/O）**：使用四条数据线进行并行数据传输，极大地提高了数据带宽。

- **优点**：
  1. **增加带宽**：由于使用了多个数据线，QSPI相比标准SPI能提供更高的数据传输速率和带宽。
  2. **适用于高速度需求的应用**：如外部存储器（Flash）的高速数据访问。
  3. **灵活的模式选择**：支持单线、双线和四线模式，可以根据需求选择合适的工作模式。