const mongoose = require("mongoose");

mongoose.connect("mongodb://localhost:27017/ecommerce", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log("MongoDB Connected"))
.catch((err) => console.error("MongoDB connection failed:", err));
const variantSchema = new mongoose.Schema({
  color: { type: String, required: true },
  size: { type: String, required: true },
  stock: { type: Number, required: true, min: 0 },
});

const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  price: { type: Number, required: true, min: 0 },
  category: { type: String, required: true },
  variants: [variantSchema], 
}, { timestamps: true });

const Product = mongoose.model("Product", productSchema);
async function insertSampleProducts() {
  await Product.deleteMany({}); 

  const products = [
    {
      name: "T-Shirt",
      price: 25,
      category: "Clothing",
      variants: [
        { color: "Red", size: "M", stock: 10 },
        { color: "Blue", size: "L", stock: 5 },
      ],
    },
    {
      name: "Sneakers",
      price: 80,
      category: "Footwear",
      variants: [
        { color: "White", size: "9", stock: 15 },
        { color: "Black", size: "10", stock: 8 },
      ],
    },
    {
      name: "Backpack",
      price: 50,
      category: "Accessories",
      variants: [
        { color: "Green", size: "Standard", stock: 20 },
      ],
    },
  ];

  const result = await Product.insertMany(products);
  console.log("Sample products inserted:", result);
}
async function runQueries() {
  const allProducts = await Product.find();
  console.log("\nAll Products:");
  console.log(allProducts);
  const clothingProducts = await Product.find({ category: "Clothing" });
  console.log("\nClothing Products:");
  console.log(clothingProducts);
  const variantsOnly = await Product.find(
    { name: "T-Shirt" },
    { name: 1, variants: 1, _id: 0 }
  );
  console.log("\nT-Shirt Variants Only:");
  console.log(variantsOnly);
  await Product.updateOne(
    { name: "T-Shirt", "variants.color": "Red", "variants.size": "M" },
    { $set: { "variants.$.stock": 7 } }
  );
  console.log("\nUpdated stock for Red T-Shirt variant");
async function main() {
  await insertSampleProducts();
  await runQueries();
  mongoose.connection.close();
}

main();
