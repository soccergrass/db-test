<script>
import { Row, Col, Collapse, Navbar, NavbarToggler, NavbarBrand, Nav, NavItem, Button, NavLink, Dropdown, DropdownToggle, DropdownMenu, DropdownItem, Form, InputGroup, Input } from 'sveltestrap'
import {site, cert, user, auth} from '../../../dir/init.js'
import MD5 from 'crypto-js/md5.js'
import { createEventDispatcher } from 'svelte'

const dispatch = createEventDispatcher()
export let isAuth
let err

async function handleSubmit(e){
    console.log(e)
    if(!document.getElementById('text').value){
        return
    }
    let useText = document.getElementById('text').value
    const stamp = Date.now()
    const sendMessage = {id: stamp + '-' + MD5(useText).toString(), stamp, replied: false, replies: false, iden: isAuth, file: '', text: useText, category: document.getElementById('category').value || null, comment: document.getElementById('comment').value || null}
    const makePost = site.get('posts').get(sendMessage.id).put(sendMessage, null, {opt: {cert}})
    if(sendMessage.category){
        site.get('categories').get(sendMessage.category).get(sendMessage.id).put(makePost, null, {opt: {cert}})
    }
    site.get('users').get(sendMessage.iden).get(sendMessage.id).put(makePost, null, {opt: {cert}})
    if(sendMessage.comment){
        site.get('comments').get(sendMessage.comment).get(sendMessage.id).put(makePost, null, {opt: {cert}})
    }
    document.getElementById('text').value = ''
    document.getElementById('category').value = ''
    document.getElementById('comment').value = ''
    // dispatch('message', sendMessage)
    return sendMessage
}
</script>

{#if err}
    <Row class="mt-4">
        <Col>
            <p>{err}</p>
        </Col>
    </Row>
{/if}

<Row class="mt-4">
    <Col>
        <Form>
            <Input type="text" placeholder="category" id="category"/>
            <Input type="textarea" placeholder="post" id="text"/>
            <Input type="text" placeholder="user" id="comment"/>
            <Button type="button" on:click={(e) => {
                handleSubmit(e).then((data) => {dispatch('message', data)}).catch((e) => {err = e.message;setTimeout(() => {err = null}, 5000);})
            }}>submit</Button>
        </Form>
    </Col>
</Row>
<style></style>