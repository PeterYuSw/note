# dma_buf

## basic
```C
struct dma_buf {
	const struct dma_buf_ops *ops;
};
```

struct dma_buf_ops {
	int (*attach)(struct dma_buf *, struct dma_buf_attachment *);
	int (*pin)(struct dma_buf_attachment *attach);
	struct sg_table *(*map_dma_buf)(struct dma_buf_attachment *, enum dma_data_direction);
};

## exporter

### exporter example driver

