<template>
    <br />
    <p>Selecione o Estado</p>
    <br />
    <select v-model="estadoSelecionado">
        <option v-for="n in estados" :key="n" :value="n.uf">{{ n.name }}</option>
    </select>
    <br />
    <p>Selecione o Município</p>
    <br />
    <select v-model="searchID">
        <option v-for="n in cities" :key="n" :value="n.id">{{ n.nome }}</option>
    </select>
    <br />
    <p>Respostas</p>
    <br />
    {{ searchID }}
    <br />
<!--     {{ informationPopCensus }}
 -->
    <div v-for=" results in informationPopCensus" :key="results">
        <p v-for=" information in results.resultados" :key="information">
            <ul>
                <li v-for=" category in information.classificacoes" :key="category">{{category.categoria}}</li>
                <li v-for=" yearQuery in information.series" :key="yearQuery">
                    <span v-for=" year in Object.keys(yearQuery.serie)" :key="year"> 
                        {{year}}: {{yearQuery.serie[year]}} <br>
                    </span>
                </li>
            </ul>
        </p>
    </div>    
    <br />
   <!--  {{ informationPopCountOne }} -->
    <div v-for=" results in informationPopCountOne" :key="results">
        <p v-for=" information in results.resultados" :key="information">
            <ul>
                <li v-for=" category in information.classificacoes" :key="category">{{category.categoria}}</li>
                <li v-for=" yearQuery in information.series" :key="yearQuery">
                    <span v-for=" year in Object.keys(yearQuery.serie)" :key="year"> 
                        {{year}}: {{yearQuery.serie[year]}} <br>
                    </span>
                </li>
            </ul>
        </p>
    </div>
    <br />
    
    <div v-for=" results in informationPopCountTwo" :key="results">
        <p v-for=" information in results.resultados" :key="information">
            <ul>
                <li v-for=" yearQuery in information.series" :key="yearQuery">
                    <span v-for=" year in Object.keys(yearQuery.serie)" :key="year"> 
                        {{year}}: {{yearQuery.serie[year]}}
                    </span>
                </li>
            </ul>
        </p>
    </div>
</template>

<script setup>
</script>

<script>
import axios from 'axios'
import Estados from '../datas/Estados'

export default {
    data() {
        return {
            allInformation: [],
            cities: [],
            estados: Estados, //lembrar sempre se criar a variável depois de importar
            searchID: "",
            estadoSelecionado: "",
            informationPopCensus: [], //Censo demográfico de 1991, 2000 e 2010.
            informationPopCountOne: [], //Contagem Populacional de 1996.
            informationPopCountTwo: [], //Contagem Populacional de 2007.
        }
    },
    watch: {
        text(novo, antigo) {
            console.log("alterou o texto para: " + novo + " / " + antigo)
        },

        estadoSelecionado() {
            this.filterCities(this.estadoSelecionado)
        },

        async searchID() {
            let ID = this.searchID.toString().trim()
            this.informationPopCensus = await this.searchPopulationCensus(ID);
            this.informationPopCountOne = await this.searchPopulationCountOne(ID);
            this.informationPopCountTwo = await this.searchPopulationCountTwo(ID);
        },
    },
    methods: {
        async getAllCities() {
            const resp = await axios.get("https://servicodados.ibge.gov.br/api/v1/localidades/municipios/")
            return resp.data
        },

        filterCities(UF) {
            this.cities = this.allInformation.filter(a => {
                return a["regiao-imediata"]["regiao-intermediaria"].UF.sigla == UF //essa é a comparação
            })
        },

        async searchPopulationCensus(searchID) {
            const resp = await axios.get(`https://servicodados.ibge.gov.br/api/v3/agregados/202/periodos/1991|2000|2010/variaveis/93?localidades=N6[${searchID}]&classificacao=1[all]`)
            return resp.data
        },
        async searchPopulationCountOne(searchID) {
            const resp = await axios.get(`https://servicodados.ibge.gov.br/api/v3/agregados/305/periodos/-6/variaveis/247?localidades=N6[${searchID}]&classificacao=1[all]`)
            return resp.data
        },
        async searchPopulationCountTwo(searchID) {
            const resp = await axios.get(`https://servicodados.ibge.gov.br/api/v3/agregados/793/periodos/-6/variaveis/93?localidades=N6[${searchID}]`)
            return resp.data
        },
    },

    async mounted() {
        this.allInformation = await this.getAllCities()
    }
}
</script>