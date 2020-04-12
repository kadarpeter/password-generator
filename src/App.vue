<template>
    <div id="app">
        <div class="formItem--password">
            <label class="formItem__title" for="newPassword">Jelszó</label>
            <div class="formItem__note">{{noteText}}</div>
            <div class="formItem__password">
                <div class="passwordStrength" :class="'str_' + strengthLevel"></div>
                <div class="inputWrapper">
                    <div @click="show = !show" class="showPassword" :class="{hidden: !show}" :title="!show ? titleOptions.show : titleOptions.hide"></div>
                    <input :type="show === true ? 'text' : 'password'" id="newPassword" placeholder="Jelszó" :character="password" v-model="password" @keyup="showPasswordCounter">
                    <div @click="onCopyPass" class="copyPassword" :title="titleOptions.copy"></div>
                </div>
            </div>
            <div class="formItem__alert">{{alertText}}</div>
            <div class="formItem__bottom">
                <div class="passwordCounter">{{password.length}}
                    <transition name="slide-fade">
                        <span v-if="counterShow"></span>
                    </transition>
                </div>
                <div class="btnWrapper">
                    <div @click="onClear" class="btn--delPassword" :title="titleOptions.del"></div>
                    <div @click="onGenerate" class="btn--generatePassword">Generálás</div>
                </div>
            </div>
        </div>
        <div class="formItem--submit">
            <input @click="onValidate" type="submit" value="Validálás" class="btn--submit">
        </div>
    </div>
</template>

<script>

export default {
    name: 'App',
    data() {
        return {
            show: false,
            counterShow: false,
            alertText: '',
            titleOptions: {
                show: 'Jelszó megjelenítése',
                hide: 'Jelszó elrejtése',
                del: 'Jelszó törlése',
                copy: 'Jelszó másolása'
            },
            password: '',
            // Set > Password length
            passwordLength: [
                {
                    minLength: 8,
                    errorMessage: 'A jelszó nem éri el a minimális karakter számot!'
                }
            ],
            // Set > Password rules
            rules: [
                {
                    name: "kisbetű",
                    character: "abcdefghijklmnopqrstuvwxyz",
                    checked: true,
                    errorMessage: 'Nem tartalmaz kisbetűt!',
                    regex: /[a-z]/
                },
                {
                    name: "nagybetű",
                    character: "ABCDEFGHIJKLMNOPQRSTUVWXYZ",
                    checked: true,
                    errorMessage: 'Nem tartalmaz nagybetűt!',
                    regex:/[A-Z]/
                },
                {
                    name: "szám",
                    character: "0123456789",
                    checked: true,
                    errorMessage: 'Nem tartalmaz számot!',
                    regex:/\d/
                },
                {
                    name: "speciális karakter",
                    character: "_-+=*&^%$#@!`[]{}()<>~|",
                    checked: true,
                    errorMessage: 'Nem tartalmaz speciális karekter!',
                    regex: /\W/
                }
            ]
        }
    },
    methods: {
        // Validate
        onValidate() {
            if (this.password.length < this.passwordLength[0].minLength) {
                this.alertText = this.passwordLength[0].errorMessage
            } else if (this.password.length >= this.passwordLength[0].minLength) {
                this.alertText = "More error"
            } else {
                this.alertText = ""
            }
        },
        // Generate Password
        onGenerate: function () {
            let result = "";
            let allowedCharacter = "";
            for (var j = 0; j < this.rules.length; j++) {
                if (this.rules[j].checked) {
                    allowedCharacter += this.rules[j].character;
                }
            }
            for ( var i = 0; i < this.passwordLength[0].minLength; i++ ) {
                result += allowedCharacter.charAt(Math.floor(Math.random() * allowedCharacter.length));
            }
            this.password = result;
            this.show = true;
        },
        // Clear Input Value
        onClear: function () {
            this.password = '';
            this.show = false;
        },
        // Copy Password
        onCopyPass: function() {
            let passwordToCopy = this.password;
            try {
                // 1) Copy text
                navigator.clipboard.writeText(passwordToCopy);
                // 2) Catch errors
            } catch (err) {
                console.error('Jelszó másolása sikertelen:', err);
            }
        },
        showPasswordCounter() {
            if (this.password.length > 0) {
                this.counterShow = true;
            } else {
                this.counterShow = false;
            }
        }
    },
    computed: {
        // Generate password note text
        noteText: function () {
            let characterText = "";
            for (var j = 0; j < this.rules.length; j++) {
                if (this.rules[j].checked) {
                    characterText += (this.rules[j].name + ', ');
                }
            }
            return ('Jelszó hossza: min. ' + this.passwordLength[0].minLength + ' karakter. Tartalmaz: ' + characterText.slice(0, -2) + '.');
        },
        // Password level
        scorePassword() {
            let score = 0;
            if (this.password === '') return score;

            let letters = {};
            for (let i = 0; i < this.password.length; i++) {
                letters[this.password[i]] = (letters[this.password[i]] || 0) + 1;
                score += 5.0 / letters[this.password[i]];
            }

            let variations = {
                digits: /\d/.test(this.password),
                lower: /[a-z]/.test(this.password),
                upper: /[A-Z]/.test(this.password),
                special: /\W/.test(this.password),
            };

            let variationCount = 0;
            for (let check in variations) {
                variationCount += (variations[check] == true) ? 1 : 0;
            }

            score += (variationCount - 1) * 10;

            return parseInt(score);
        },
        strengthLevel() {
            let pass = this.scorePassword;
            if(pass === 0) return 0;
            if(pass < 20) return 1;
            if(pass < 40) return 2;
            if(pass < 60) return 3;
            if(pass < 80) return 4;
            if(pass >= 80) return 5;
        }
    },
    components: {

    }
}
</script>

<style>
    .slide-fade-enter-active {
        transition: all .3s ease;
    }
    .slide-fade-leave-active {
        transition: all .125s ease-out;
    }
    .slide-fade-enter, .slide-fade-leave-to {
        transform: translateX(10px);
        opacity: 0;
    }
</style>
