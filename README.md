st.write(dataframe)
    data_quality_check = st.checkbox('Request Data Quality Check')
    
    if data_quality_check:
        st.write("The following data quality analysis has been made")
        st.markdown("**1. The dataset column names have been checked for trailing spaces**")
        trailing_spaces = dataframe.columns[dataframe.columns.str.contains("\s+$", regex=True)]
        if trailing_spaces.empty:
            st.markdown('*Columns_ names_ are_ found_ ok*')
        else:
            st.markdown("*Columns with trailing spaces:*")
            st.write(f"{', '.join(trailing_spaces)}")

        # Check data type of columns with name 'date'
        st.markdown("**2. The dataset's date columns have been checked for the correct data type**")
        date_cols = dataframe.select_dtypes(include="object").filter(regex="(?i)date").columns
        if date_cols.empty:
            st.markdown('*There are no Columns with date*')
        else:
            for col in date_cols:
                if pd.to_datetime(dataframe[col], errors="coerce").isna().sum() > 0:
                    st.markdown("*Column {col} should contain dates but has wrong data type*")
                else:
                    st.markdown("*Columns with date are of the correct data type:*")
                    st.write(f"{', '.join(date_cols)}")
        
        st.markdown("**:red[CSV BOT recommends fixing data quality issues prior to querying your data]**")


        # download analyst_report based on  pre-defined queries

        # df = pd.read_csv(uploaded_file)
                #s = pd.read_csv(doc)
            #st.subheader(" Download the file")
            #st.download_button('Download Docx',data=st.session_state["generated"], file_name='New_File.docx')
        
        agent = create_pandas_dataframe_agent(
            ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613"),
            dataframe,
            verbose=True,
            agent_type=AgentType.OPENAI_FUNCTIONS
           # agent_executor_kwargs={"handle_parsing_error" : True} #errror resolving out parsing 
        )
        document = Document()
        style = document.styles['Normal']
        font = style.font
        font.name = 'Calibri'
        font.size = Pt(12)
        document.add_heading('DataGenieSuite', 0)
        document.add_picture('attackImg.png', width = Inches(1))


        p1= document.add_heading('\n \n Detailed Analysis Report on the Source Data \n ',level=2)

        # Call the agent with relevant prompts and write to output document

        p2= document.add_paragraph('\n Total Number of records in source file')
        val = agent.run(f"""
        You are an Data Analyst \
        you are assigned the task to analyse the data and come up with best data model for target DB \
        the input is an insurance data  \
        return the result in tabular fomat \
        how many rows are there in total
        """)
        document.add_paragraph('\t' + val)

        p3= document.add_paragraph('\n List of Columns and its datatype in source File')
        val1 = agent.run("List the columns  present, column names along with the data type")
        document.add_paragraph('\t' + val1)


        p4= document.add_paragraph('\n Suggested Data Model for the Source data ')
        val2 =agent.run(f"""
        You are an Data Analyst \
        you are assigned the task to analyse the data and come up with best data model for target DB \
        the input is an insurance data  \
        you have to find the correralation of the data with agency details and give the best suited data model
        """)
        document.add_paragraph('\t' + val2)


        name='DataAnalysisReport_insurance'
        #document.save(f'{name}.docx')

        buffer = io.BytesIO()
        document.save(buffer)
        buffer.seek(0)
        st.download_button(
            label="Download Data Analysis Report",
            data=buffer,
            file_name=f'{name}.docx',
            mime='docx',
        )
