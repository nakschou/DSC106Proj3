<script>
    export function getCompanyInfo(companyName, data) {
        for (let i = 0; i < data.length; i++) {
            const companyData = data[i]; // Assuming the keys are numbers
            if (companyData['Company Name'] === companyName) {
                return companyData;
            }
        }
        //return null; // Company not found
    }
</script>