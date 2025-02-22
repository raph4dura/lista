<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lista de Alimentos Naturais</title>
    <style>
        /* Reset and base styles */
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: Arial, sans-serif;
            background-color: #000;
            color: #fff;
            padding: 20px;
        }

        .container {
            max-width: 1200px;
            margin: 20px auto;
            background-color: #000;
            padding: 20px;
            border: 1px solid #fff;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
        }

        .foods-list {
            padding: 20px;
        }

        .foods-list h2 {
            margin-top: 0;
        }

        ul {
            list-style-type: none;
            padding: 0;
        }

        li {
            margin-bottom: 8px;
            padding: 10px;
            border-radius: 4px;
            background-color: #333;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        li:hover {
            background-color: #444;
        }

        .add-btn,
        .control-btn {
            padding: 8px 16px;
            background-color: #007bff;
            color: #fff;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .add-btn:hover,
        .control-btn:hover {
            background-color: #0056b3;
        }

        .added-foods-container {
            position: fixed;
            top: 20px;
            right: 20px;
            bottom: 20px;
            width: 300px;
            background-color: #000;
            border: 1px solid #fff;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
            overflow-y: auto;
            display: none;
            z-index: 1000;
            color: #fff;
        }

        .added-foods {
            padding: 20px;
        }

        .added-foods h2 {
            margin-top: 0;
        }

        .btn-send-email {
            margin-top: 20px;
            padding: 10px 20px;
            background-color: #007bff;
            color: #fff;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .btn-send-email:hover {
            background-color: #0056b3;
        }

        #observationText {
            width: 100%;
            padding: 8px;
            font-size: 14px;
            border: 1px solid #fff;
            border-radius: 4px;
            resize: vertical;
            color: #000;
        }

        .quantity-controls {
            margin-top: 4px;
        }

        /* Responsive adjustments */
        @media (max-width: 768px) {
            .container {
                padding: 10px;
            }

            .added-foods-container {
                width: 100%;
                max-width: 100%;
                left: 0;
                right: 0;
                border-radius: 0;
            }
        }
    </style>
</head>
<body>
    <div id="app">
        <div class="container">
            <div class="foods-list">
                <h2>Lista de Alimentos</h2>
                <div v-for="(category, categoryName) in foodCategories" :key="categoryName">
                    <h3>{{ categoryName }}</h3>
                    <ul>
                        <li v-for="item in category" :key="item.name">
                            {{ item.name }}
                            <button class="add-btn" @click="addItem(item, categoryName)">Adicionar</button>
                        </li>
                    </ul>
                </div>
            </div>
        </div>

        <!-- Itens adicionados -->
        <div class="added-foods-container" v-show="Object.keys(addedItemsByCategory).some(category => addedItemsByCategory[category].length > 0)">
            <div class="added-foods">
                <h2>Itens Adicionados</h2>
                <template v-for="(items, category) in addedItemsByCategory" :key="category">
                    <h3>{{ category }}</h3>
                    <ul>
                        <li v-for="item in items" :key="item.name" id="item-{{ item.name }}">
                            {{ item.name }} - {{ item.quantity }}
                            <div class="quantity-controls">
                                <button class="control-btn" @click="adjustQuantity(item, category, 1)">+1</button>
                                <button class="control-btn" @click="adjustQuantity(item, category, -1)">-1</button>
                                <!-- Campo de entrada para adicionar quantidade -->
                                <input type="number" v-model.number="item.quantityToAdd" style="width: 50px;">
                                <button class="control-btn" @click="increaseQuantityByInput(item)">Adicionar</button>
                                <!-- Botões -6 e +6 -->
                                <button class="control-btn" @click="decreaseBySix(item)" v-if="item.quantity > 6">-6</button>
                                <button class="control-btn" @click="increaseBySix(item)">+6</button>
                                <!-- Botão +6 para adicionar 6 unidades -->
                                <button class="control-btn" @click="increaseBySixItem(item)">+6 Item</button>
                            </div>
                        </li>
                    </ul>
                </template>
            </div>
        </div>

        <!-- Formulário de envio por email -->
        <div>
            <h2>Enviar por Email</h2>
            <form id="emailForm" action="/send-email" method="POST">
                <textarea id="emailContent" name="emailContent" hidden></textarea>
                <div class="added-items-list">
                    <h3>Itens Adicionados</h3>
                    <ul>
                        <template v-for="(items, category) in addedItemsByCategory" :key="category">
                            <h4>{{ category }}</h4>
                            <li v-for="item in items" :key="item.name">
                                {{ item.name }} - {{ item.quantity }}
                            </li>
                        </template>
                    </ul>
                </div>
                <div class="observation">
                    <label for="observationText">Observação:</label><br>
                    <textarea id="observationText" name="observationText" rows="4" cols="50"></textarea><br><br>
                </div>
                <button type="submit" class="btn-send-email" @click.prevent="enviarPorEmail">Enviar</button>
            </form>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
    <script>
        new Vue({
            el: '#app',
            data() {
                return {
                    foodCategories: {
                        Naturais: [
                            { name: 'Maçã' },
                            { name: 'Banana' }
                        ],
                        Molhos: [
                            { name: 'Ketchup' },
                            { name: 'Mostarda' }
                        ],
                        Sais: [
                            { name: 'Sal de Mesa' },
                            { name: 'Sal Marinho' }
                        ],
                        Tags: [
                            { name: 'Orgânico', quantity: 6 },
                            { name: 'Sem Glúten', quantity: 6 }
                        ]
                    },
                    addedItems: []
                };
            },
            computed: {
                addedItemsByCategory() {
                    const categorizedItems = {
                        Naturais: [],
                        Molhos: [],
                        Sais: [],
                        Tags: []
                    };

                    this.addedItems.forEach(item => {
                        if (categorizedItems[item.category]) {
                            categorizedItems[item.category].push(item);
                            // Ordenar os itens dentro da categoria
                            categorizedItems[item.category].sort((a, b) => a.name.localeCompare(b.name));
                        }
                    });

                    return categorizedItems;
                }
            },
            methods: {
                addItem(item, category) {
                    const existingItem = this.addedItems.find(
                        added => added.name === item.name && added.category === category
                    );

                    if (existingItem) {
                        // Scroll suave para o item na lista de itens adicionados
                        const listItem = document.getElementById(`item-${existingItem.name}`);
                        if (listItem) {
                            listItem.scrollIntoView({ behavior: 'smooth', block: 'center' });
                        }
                    } else {
                        let initialQuantity = 1;
                        if (category === 'Tags') {
                            initialQuantity = 6; // Iniciar itens da categoria Tags com 6
                        }
                        const newItem = { name: item.name, category, quantity: initialQuantity };
                        
                        // Adicionar campos para quantidade a ser adicionada/subtraída
                        newItem.quantityToAdd = 1;
                        newItem.quantityToSubtract = 1;

                        this.addedItems.push(newItem);

                        // Ordenar os itens novamente após adicionar um novo item
                        this.addedItems.sort((a, b) => {
                            if (a.category === b.category) {
                                return a.name.localeCompare(b.name);
                            }
                            return a.category.localeCompare(b.category);
                        });

                        // Scroll suave para o novo item adicionado
                        setTimeout(() => {
                            const newItemElement = document.getElementById(`item-${newItem.name}`);
                            if (newItemElement) {
                                newItemElement.scrollIntoView({ behavior: 'smooth', block: 'center' });
                            }
                        }, 100); // Tempo de espera para garantir que o item foi renderizado
                    }
                },
                adjustQuantity(item, category, change) {
                    const addedItem = this.addedItems.find(
                        added => added.name === item.name && added.category === category
                    );

                    if (addedItem) {
                        addedItem.quantity += change;
                    }
                },
                increaseQuantityByInput(item) {
                    const addedItem = this.addedItems.find(
                        added => added.name === item.name && added.category === item.category
                    );

                    if (addedItem) {
                        addedItem.quantity += addedItem.quantityToAdd;
                        addedItem.quantityToAdd = 1; // Resetar campo de entrada
                    }
                },
                decreaseBySix(item) {
                    const addedItem = this.addedItems.find(
                        added => added.name === item.name && added.category === item.category
                    );

                    if (addedItem && addedItem.quantity > 6) {
                        addedItem.quantity -= 6;
                    }
                },
                increaseBySix(item) {
                    const addedItem = this.addedItems.find(
                        added => added.name === item.name && added.category === item.category
                    );

                    if (addedItem) {
                        addedItem.quantity += 6;
                    }
                },
                increaseBySixItem(item) {
                    const addedItem = this.addedItems.find(
                        added => added.name === item.name && added.category === item.category
                    );

                    if (addedItem) {
                        addedItem.quantity += 6;
                    }
                },
                enviarPorEmail() {
                    const observation = document.getElementById('observationText').value.trim();

                    let emailContent = '';
                    Object.keys(this.addedItemsByCategory).forEach(category => {
                        if (this.addedItemsByCategory[category].length > 0) {
                            emailContent += `${category}:\n`;
                            this.addedItemsByCategory[category].forEach(item => {
                                emailContent += `${item.name}: ${item.quantity}\n`;
                            });
                        }
                    });

                    if (observation !== '') {
                        emailContent += `\nObservação:\n${observation}`;
                    }

                    document.getElementById('emailContent').value = emailContent;
                    document.getElementById('observation').value = observation;

                    document.getElementById('emailForm').submit();
                }
            }
        });
    </script>
</body>
</html>
