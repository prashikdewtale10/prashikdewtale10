### Hi there 👋 this is Prashik Dewtale



import justpy as jp

def radio_demo():
    wp = jp.QuasarPage(dark=True)

    radio_group = jp.QRadioGroup(v_model='option1')
    
    radio_option1 = jp.QRadio(label='Option 1', value='option1')
    radio_option2 = jp.QRadio(label='Option 2', value='option2')
    radio_option3 = jp.QRadio(label='Option 3', value='option3')
    
    radio_group.add_child(radio_option1)
    radio_group.add_child(radio_option2)
    radio_group.add_child(radio_option3)

    wp.add_child(radio_group)
    
    return wp

jp.justpy(radio_demo)




