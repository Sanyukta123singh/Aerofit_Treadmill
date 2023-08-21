# Analysing basic metrics

data.head()
data.columns
data.shape
data.drop_duplicates()
data.info()
data.describe()
data.nunique()
data['Product'].unique()
data['Gender'].unique()
data['Age'].unique()
data['Education'].unique()

# Missing Data Analysis

def MissingValues(data):
  total_null = data.isnull().sum().sort_values(ascending = False)
  percent = ((data.isnull().sum()/data.isnull().count())*100).sort_values(ascending = False)

  print("Total records = ", data.shape[0])

  Combined = pd.concat([total_null,percent.round(2)],axis = 1,keys = ['Total Missing','In Percent'])

  return Combined

MissingValues(data)

data['Product'] = data['Product'].astype("category")
data['Gender'] = data['Gender'].astype("category")
data['MaritalStatus'] = data['MaritalStatus'].astype("category")

bins = [14,20,30,40,60]
labels = ["Teens","20s","30s","Above 40s"]
data['AgeGroup'] = pd.cut(data['Age'], bins)
data['AgeCategory'] = pd.cut(data['Age'], bins, labels=labels)

bins_income = [29000, 35000, 60000, 85000,105000]
labels_income = ['Low Income','Lower-middle income','Upper-Middle income', 'High income']
data['IncomeSlab'] = pd.cut(data['Income'], bins_income, labels = labels_income)
data.head()

# Numerical Variables - Outlier detection

def outlier_detect(df,colname,nrows=2,mcols=2,width=20,height=15):
    fig , ax = plt.subplots(nrows,mcols,figsize=(width,height))
    fig.set_facecolor("lightgrey")
    rows = 0
    for var in colname:
        ax[rows][0].set_title("Boxplot for Outlier Detection ", fontweight="bold")
        plt.ylabel(var, fontsize=12)
        sns.boxplot(y = df[var],color='m',ax=ax[rows][0])
        sns.distplot(df[var],color='m',ax=ax[rows][1])
        ax[rows][1].axvline(df[var].mean(), color='r', linestyle='--', label="Mean")
        ax[rows][1].axvline(df[var].median(), color='g', linestyle='-', label="Median")
        ax[rows][1].axvline(df[var].mode()[0], color='royalblue', linestyle='-', label="Mode")
        ax[rows][1].set_title("Outlier Detection ", fontweight="bold")
        ax[rows][1].legend({'Mean':df[var].mean(),'Median':df[var].median(),'Mode':df[var].mode()})
        rows += 1
    plt.show()


col_num = ['Income','Miles']
outlier_detect(data,col_num,2,2,14,12)

data_copy = data.copy()

Q3 = data_copy['Income'].quantile(0.75)
Q1 = data_copy['Income'].quantile(0.25)
IQR = Q3-Q1
data_copy = data_copy[(data_copy['Income'] > Q1 - 1.5*IQR) & (data_copy['Income'] < Q3 + 1.5*IQR)]
plt.show()

Q3 = data_copy['Miles'].quantile(0.75)
Q1 = data_copy['Miles'].quantile(0.25)
IQR = Q3-Q1
data_copy = data_copy[(data_copy['Miles'] > Q1 - 1.5*IQR) & (data_copy['Miles'] < Q3 + 1.5*IQR)]
plt.show()

col_num = ['Income','Miles']
outlier_detect(data_copy,col_num,2,2,14,12)

# Categorical variable Uni-variate Analysis

def cat_analysis(df, colnames, nrows=2,mcols=2,width=20,height=30, sortbyindex=False):
    fig , ax = plt.subplots(nrows,mcols,figsize=(width,height))
    fig.set_facecolor(color = 'white')
    string = "Frequency of "
    rows = 0
    for colname in colnames:
        count = (df[colname].value_counts(normalize=True)*100)
        string += colname + ' in (%)'
        if sortbyindex:
                count = count.sort_index()
        count.plot.bar(color=sns.color_palette("crest"),ax=ax[rows][0])
        ax[rows][0].set_ylabel(string, fontsize=14)
        ax[rows][0].set_xlabel(colname, fontsize=14)
        count.plot.pie(colors = sns.color_palette("crest"),autopct='%0.0f%%',
                       textprops={'fontsize': 14},ax=ax[rows][1])
        string = "Frequency of "
        rows += 1

cat_colnames = ['Product', 'Gender', 'MaritalStatus', 'AgeGroup', 'AgeCategory','IncomeSlab','Fitness']
cat_analysis(data,cat_colnames,7,2,14,40)

# Bi-Variate Analysis

def cat_bi_analysis(df,colname,depend_var,nrows=2,mcols=2,width=20,height=15):
    fig , ax = plt.subplots(nrows,mcols,figsize=(width,height))
    sns.set(style='white')
    rows = 0
    string = " based Distribution"
    for var in colname:
        string = var + string
        sns.countplot(data=df,x=depend_var, hue=var, palette="hls",ax=ax[rows][0])
        sns.countplot(data=df, x=var, hue=depend_var, palette="husl",ax=ax[rows][1])
        ax[rows][0].set_title(string, fontweight="bold",fontsize=14)
        ax[rows][1].set_title(string, fontweight="bold",fontsize=14)
        ax[rows][0].set_ylabel('count', fontweight="bold",fontsize=14)
        ax[rows][0].set_xlabel(var,fontweight="bold", fontsize=14)
        ax[rows][1].set_ylabel('count', fontweight="bold",fontsize=14)
        ax[rows][1].set_xlabel(var,fontweight="bold", fontsize=14)
        rows += 1
        string = " based Distribution"
    plt.show()

col_names = ['Gender', 'MaritalStatus', 'AgeGroup', 'AgeCategory','IncomeSlab','Fitness','Education']
cat_bi_analysis(data,col_names,'Product',7,2,20,45)

# Bivariate Analysis for Numerical variables

def num_mult_analysis(df,colname,category,groupby,nrows=2,mcols=2,width=20,height=15):
    fig , ax = plt.subplots(nrows,mcols,figsize=(width,height))
    sns.set(style='white')
    fig.set_facecolor("lightgrey")
    rows = 0
    for var in colname:
        sns.boxplot(x = category,y = var, hue = groupby,data = df,ax=ax[rows][0])
        sns.pointplot(x=df[category],y=df[var],hue=df[groupby],ax=ax[rows][1])
        ax[rows][0].set_ylabel(var, fontweight="bold",fontsize=14)
        ax[rows][0].set_xlabel(category,fontweight="bold", fontsize=14)
        ax[rows][1].set_ylabel(var, fontweight="bold",fontsize=14)
        ax[rows][1].set_xlabel(category,fontweight="bold", fontsize=14)
        rows += 1
    plt.show()

col_num = ['Income', 'Miles']
num_mult_analysis(data,col_num,"AgeCategory","Product")

plt.figure(figsize = (16,10))
sns.heatmap(data.corr(), annot=True, vmin = -1, vmax = 1, cmap = "YlGnBu")
plt.show()

sns.pairplot(data,hue = "Product")
plt.show()

# Analysis using Contingency Tables to Calculate Probabilities

pd.crosstab(index = data['Product'],columns = [data['IncomeSlab']], margins=True)
pd.crosstab(index = data['Product'],columns = [data['Gender']],margins=True)
pd.crosstab(index=data['Product'],columns=[data['Fitness']],margins=True)
pd.crosstab(index=data['Product'],columns=[data['AgeCategory']],margins=True)
pd.crosstab(index=data['Product'],columns=[data['MaritalStatus']],margins=True)
