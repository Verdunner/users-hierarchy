<template>
    <div class="app">
        <!-- Левая колонка -->
        <div class="app__user-list">
            <h2>Пользователи</h2>
            <button @click="showModal = !showModal">Добавить</button>

            <div class="app__user-table">
                <div class="app__user-row app__user-row--header">
                    <div
                        class="app__cell app__cell--id"
                        @click="changeSort('id')"
                    >
                        ID
                        <span v-if="sortKey === 'id'">{{
                            sortOrder === 'asc' ? '↑' : '↓'
                        }}</span>
                    </div>
                    <div
                        class="app__cell app__cell--name"
                        @click="changeSort('name')"
                    >
                        Имя
                        <span v-if="sortKey === 'name'">{{
                            sortOrder === 'asc' ? '↑' : '↓'
                        }}</span>
                    </div>
                    <div
                        class="app__cell app__cell--phone"
                        @click="changeSort('phone')"
                    >
                        Телефон
                        <span v-if="sortKey === 'phone'">{{
                            sortOrder === 'asc' ? '↑' : '↓'
                        }}</span>
                    </div>
                </div>

                <div
                    class="app__user-row"
                    v-for="user in sortedUsers"
                    :key="user.id"
                >
                    <div class="app__cell app__cell--id">{{ user.id }}</div>
                    <div
                        class="app__cell app__cell--name"
                        :style="{
                            paddingLeft: user.level * 30 + 'px',
                            marginRight: user.level * -30 + 'px',
                        }"
                    >
                        {{ user.name }}
                    </div>
                    <div class="app__cell app__cell--phone">
                        {{ user.phone }}
                    </div>
                </div>
            </div>
        </div>

        <!-- Правая колонка -->
        <div class="app__modal-area">
            <div v-if="showModal" class="app__modal">
                <h2>Добавить пользователя</h2>
                <form @submit.prevent="addUser">
                    <div class="app__form-group">
                        <label>Имя:</label>
                        <input v-model="newName" type="text" required />
                    </div>
                    <div class="app__form-group">
                        <label>Телефон:</label>
                        <input
                            v-model="newPhone"
                            type="tel"
                            placeholder="+7 941 123 21 42"
                            required
                        />
                    </div>
                    <div class="app__form-group">
                        <label>Начальник:</label>
                        <select v-model="newManagerId">
                            <option :value="null">Без начальника</option>
                            <option
                                v-for="user in users"
                                :key="user.id"
                                :value="user.id"
                            >
                                {{ user.name }}
                            </option>
                        </select>
                    </div>
                    <button type="submit">Добавить</button>
                    <button type="button" @click="showModal = false">
                        Закрыть
                    </button>
                </form>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'App',
    data() {
        return {
            showModal: true,
            sortKey: 'id',
            sortOrder: 'desc',
            users: [
                {
                    id: 1,
                    name: 'Марина',
                    phone: '+7 941 123 21 42',
                    managerId: null,
                },
                {
                    id: 2,
                    name: 'Петр',
                    phone: '+7 941 123 21 42',
                    managerId: null,
                },
                {
                    id: 3,
                    name: 'Алексей',
                    phone: '+7 921 555 78 90',
                    managerId: 1,
                },
                {
                    id: 4,
                    name: 'Иван',
                    phone: '+7 921 555 78 90',
                    managerId: 3,
                },
            ],
            sortedUsers: [],
            newName: '',
            newPhone: '',
            newManagerId: null,
        };
    },
    methods: {
        changeSort(key) {
            if (this.sortKey === key) {
                this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
            } else {
                this.sortKey = key;
                this.sortOrder = 'asc';
            }
            this.sortUsers();
        },
        sortUsers() {
            this.sortedUsers = [];

            const sortFn = (a, b) => {
                let valA = a[this.sortKey];
                let valB = b[this.sortKey];

                // Для строк — сделать toLowerCase
                if (typeof valA === 'string') valA = valA.toLowerCase();
                if (typeof valB === 'string') valB = valB.toLowerCase();

                if (valA < valB) return this.sortOrder === 'asc' ? -1 : 1;
                if (valA > valB) return this.sortOrder === 'asc' ? 1 : -1;
                return 0;
            };

            // Рекурсивная функция обхода по менеджерам
            const addUsersByManager = (managerId, level) => {
                this.users
                    .filter((user) => user.managerId === managerId)
                    .sort(sortFn)
                    .forEach((user) => {
                        // Копируем пользователя и добавляем уровень вложенности
                        this.sortedUsers.push({ ...user, level });
                        // Рекурсивно добавляем подчиненных текущего пользователя
                        addUsersByManager(user.id, level + 1);
                    });
            };
            // Запускаем с начальников (managerId === null)
            addUsersByManager(null, 0);
        },
        addUser() {
            // Определяем новый ID — максимально существующий + 1
            const newId = this.users.length
                ? Math.max(...this.users.map((u) => u.id)) + 1
                : 1;

            this.users.push({
                id: newId,
                name: this.newName,
                phone: this.newPhone,
                managerId: this.newManagerId,
            });

            localStorage.setItem('users', JSON.stringify(this.users));

            this.sortUsers();

            // Очистка формы
            this.newName = '';
            this.newPhone = '';
            this.newManagerId = null;
        },
    },
    mounted() {
        // Загружаем из localStorage, если есть
        const savedUsers = localStorage.getItem('users');
        if (savedUsers) {
            this.users = JSON.parse(savedUsers);
        } else {
            // Иначе можно подгрузить дефолтный массив (если он в data задан)
            this.users = [
                {
                    id: 1,
                    name: 'Марина',
                    phone: '+7 941 123 21 42',
                    managerId: null,
                },
                {
                    id: 2,
                    name: 'Петр',
                    phone: '+7 941 123 21 42',
                    managerId: null,
                },
                {
                    id: 3,
                    name: 'Алексей',
                    phone: '+7 921 555 78 90',
                    managerId: 1,
                },
                {
                    id: 4,
                    name: 'Иван',
                    phone: '+7 921 555 78 90',
                    managerId: 3,
                },
            ];
        }
        this.sortUsers();
    },
};
</script>

<style scoped>
.app {
    display: flex;
    gap: 20px;
    margin: -8px;
    padding: 20px 100px;
    height: 100vh;
    box-sizing: border-box;
    background-color: slategray;
}

.app__user-list {
    flex: 0 0 50%;
}

.app__user-table {
    margin-top: 10px;
}

.app__user-row {
    display: flex;
    border: 1px solid #ccc;
    padding: 6px 0;
}

.app__user-row--header {
    font-weight: bold;
    border: none;
    padding-bottom: 8px;
}

.app__cell {
    padding: 0 8px;
}

.app__cell--id {
    flex: 1;
}

.app__cell--name {
    flex: 2;
}

.app__cell--phone {
    flex: 3;
}

.app__modal-area {
    flex: 1;
    position: relative;
}

.app__modal {
    position: absolute;
    top: 40px;
    left: 40px;
    right: 40px;
    padding: 20px;
    background: silver;
    border: 1px solid #ccc;
    box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
    z-index: 10;
}

.app__form-group {
    margin-bottom: 10px;
    display: flex;
    flex-direction: column;
}

input {
    padding: 4px 8px;
    font-size: 14px;
}
</style>
